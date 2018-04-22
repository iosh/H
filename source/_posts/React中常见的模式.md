---
title: React中常见的模式
date: 2018-04-22 16:13:06
tags: React
---

[React in patterns](https://krasimir.gitbooks.io/react-in-patterns/content/)   观看笔记

# 通讯

每个 React 组件就像一个独立运行的小系统，他有自己的输入和输出，分别讲一下

## 输入

在 React 中是通过 props 进行输入的，就像下面这样的

```jsx
function Title(props) {
  return <h1>{ props.text }</h1>;
}

// 下面这段是定义 props 类型
Title.propTypes = {
  text: PropTypes.string
};

// 下面这段定了 props 的默认值 
Title.defaultProps = {
  text: 'Hello world'
};

// App.jsx
function App() {
    // 如果这里传了 text 属性，那么就会覆盖上面定义的 text: 'Hello world'，如果没定义 就会text属性就会默认被定义为 text: 'Hello world'
  return <Title text='Hello React' />;
}
```

这个`Title`组件只有一个 props 属性`text` ，父组件 `App` 在调用 `<Title/>`组件的时候需要提供一个 `text`属性，除了在组件调用的时候提供 `text`属性之外，还应该在`propsType`中定义 `text`的类型检查，这一步很少有人写，我也没怎么写过，但是在多人协作的情况下显得就比较重要了，在多人协作情况下，如果给了错误的类型属性会导致直接抛出错误，这里还有一个`defaultProps`这是另外一个比较有用的，可以用它来设置默认的`props`的值，当调用组件的时候忘了传递相关属性，那么这个默认值显得就比较有用了(学习了)。

在 React 中没有严格的限制 props 传递的内容，这使得我们可以做一些有意思的事情`将一个组件作为参数传递`

```jsx
function SomethingElse({ answer }) {
  return <div>The answer is { answer }</div>;
}
function Answer() {
  return <span>42</span>;
}

// later somewhere in our application
<SomethingElse answer={ <Answer /> } />
```

React 还有一个 `props.children` 属性，它代表可以访问父组件传递的子组件，例如这样

```jsx
function Title({ text, children }) {
  return (
    <h1>
      { text }
       {/*这里可以拿到父组件传递下来的子组件span */}
      { children }
    </h1>
  );
}
function App() {
  return (
    <Title text='Hello React'>
      <span>community</span>
    </Title>
  );
}
```

## 输出

React 里面最明显的输出就是显示一个 `HTML`，在视觉上，这也是我们想得到的，但是 `props`可能是任何东西，一个函数、一个字符串、一个数组、等等等等

在下面例子中，组件接受一个回调函数，将 `input`的输入通过回调函数输出出去，传递给 `App`组件

```jsx
function NameField({ valueUpdated }) {
  return (
    <input
      onChange={event => valueUpdated(event.target.value) } />
  );
};
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { name: '' };
  }
  render() {
    return (
      <div>
        <NameField
          valueUpdated={ name => this.setState({ name }) } />
        Name: { this.state.name }
      </div>
    );
  }
};
```

在程序中，经常需要一个逻辑入口点，在`React`中带有一些方便便捷的`声明周期方法`，可以在不同的时间做一些不同的事情，例如可以在生命周期中触发请求，获得数据资源。

```jsx
class ResultsPage extends React.Component {
  componentDidMount() {
      // 触发请求获得数据
    this.props.getResults();
  }
  render() {
    if (this.props.results) {
      return <List results={ this.props.results } />;
    } else {
      return <LoadingScreen />
    }
  }
}
```

可以将每个 React 组件视为一个黑盒，它有自己的输入输出和声明周期，将由开发者来组成这些盒子，这也是 React 提供的诸多优势之一，易于抽象和撰写。

# 事件处理

React 为事件处理提供了一系列属性，该解决方案与标准 DOM 中使用的解决方案几乎相同，但是也有一些差异，比如使用驼峰写法来添加一个事件，但是总体来说还是非常相似的。(这些所有的驼峰写法的事件，都是经过 React 包装的，在这些事件里面调用 this.setState 都是异步的，就是多次的 setState都会被集中执行)

```jsx
const theLogoIsClicked = () => alert('Clicked');

<Logo onClick={ theLogoIsClicked } />
<input
  type='text'
  onChange={event => theInputIsChanged(event.target.value) } />
```

这样通常我们在包含事件的元素组件中处理事件，就像下面那样，有一个点击事件，每次点击都是运行相同的函数

```jsx
class Switcher extends React.Component {
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
    // 在JavaScript中没有私有方法这一说，所以有些喜欢加一个下划线来表明方法是私有的，但是在函数名中加下划线也可能过不去eslint代码检查
  _handleButtonClick() {
    console.log('Button is clicked');
  }
};
```

这样，`_handleButtonClick`是一个函数，那么函数中的`this`指向哪里？严格模式下是`undefined`非严格模式下函数内部的`this`指向`window`对象，所以要在函数内部使用`this`需要做点额外的事情具体代码如下

```jsx
// 第一种方案使用 bind
<button onClick={ this._handleButtonClick.bind(this) }>
  click me
</button>
```

但是`bind`之后每一次点击都会重新生成一个新的函数，更好的方法是在构造函数中绑定，这样只会生成一次

```jsx
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
      // 在构造函数中进行 bind
    this._buttonClick = this._handleButtonClick.bind(this);
  }
  render() {
    return (
      <button onClick={ this._buttonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
  }
};
```

这样的话，又会有另外一个问题，如果有很多事件函数，难道一个一个的在构造函数里面`bind`？那么可以使用下面这种提案阶段语法。

```jsx
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
  }
  render() {
    return (
      <button onClick={ this._buttonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick = () => {
    console.log(`Button is clicked inside ${ this.state.name }`);
  }
};
```

这样写 `函数名 = () => {}` 这是一种提案阶段语法，但是已经在 `Facebook`内部大量使用，`Facebook`承诺如果将来这种语法没能称为标准(板上钉钉会上标准，除非发生灵异事件)，那么会由`Facebook`提供babel插件让开发者可以继续使用。



# 组合

React 最大的好处之一就是组合性，举个简单例子，假设有一个程序，有三个 React 组件，`App` 嵌套 `Header` 嵌套 `Navigation`组件

那么组合这些组件的简单方法就是在合适的地方引用它们

```jsx
// app.jsx
import Header from './Header.jsx';

export default function App() {
  return <Header />;
}

// Header.jsx
import Navigation from './Navigation.jsx';

export default function Header() {
  return <header><Navigation /></header>;
}

// Navigation.jsx
export default function Navigation() {
  return (<nav> ... </nav>);
}
```

按照这种方法进行组合，会有几个问题

1. 可能会让别人误认为 `App` 组件实现了所有功能，没有其他组建了。
2. 这样写代码很难测试，加入每个组件都有一部分业务逻辑，为了分别测试他们，但是由于这个组件本身引用了另一个组件，这样嵌套的模式，让测试极为繁重。

## 使用 React 的 children API

在 React 中，有一个很方便的 `children`API，通过它可以直接从 props 中读取父组件给他的子组件。

```jsx
export default function App() {
  return (
    <Header>
      <Navigation />
    </Header>
  );
}
export default function Header({ children }) {
  return <header>{ children }</header>;
};
```

这样写组件可以很方便的测试`Header`组件，将`Header`组件和其他组件隔离开来，不依赖任何其他组件。

## 传递一个子组件作为Porps

每个 React 组件都会收到 props，对 props 是什么类型，React 并没有任何严格的规定，所以可以让我们传递组件作为 props

```jsx
const Title = function () {
  return <h1>Hello there!</h1>;
}
const Header = function ({ title, children }) {
  return (
    <header>
      { title }
      { children }
    </header>
  );
}
function App() {
  return (
    <Header title={ <Title /> }>{/* 给title属性传递一个组件作为属性 */}
      <Navigation />
    </Header>
  );
};
```

给一个属性传递一个组件作为属性值，这种技术非常好用，考虑一个情况，有一个组件可以被多次复用，但是每个引用该组件的地方都需要做一点变化，那么将这个变化的部分写成一个属性，属性接受一个组件作为值。就可以很好的进行复用了。

# 高阶组件

长期一来，高阶组件作为增强 React 组件最流行的方式，运作方式看起来非常像装饰器(关于装饰器，Create-react-app 中为什么不支持，在文档中官方表示说装饰器还没有上标准，而且 babel 也没有正式实现，意思就是还在试验阶段，所以最终装饰器会是什么样子，还不确定，以至于 Facebook 官方并没有采用装饰器，而且装饰器没有办法提供模拟脚本，所以一旦最终的行为与现在不一致，他们将没有办法为用户提供过度。以至于一直不支持)

在技术方面，高阶组件通常接受我们的原始组件，然后返回增强后的函数或者组件，最小例子如下：

```jsx
// 2. 定义一个函数，函数接受一个组件作为参数，并且返回一个添加了 props 属性的组件。
var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component {...this.props} />
      )
    }
  };

var OriginalTitle = () => <h1>Hello world</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
// 1. 定义一个普通的 App 组件
class App extends React.Component {
  render() {
    return <EnhancedTitle />;
  }
};
```

高阶组件做的第一件事，就是渲染初始组件，代理传递`props` 给它是一种很好的做法，这样我们将保留原始组件的输入，这就是这种模式的第一大优势---因为我们控制组件的输入，所以我们可能会发送组件通常无法访问的内容，假设我们有一个`OriginalTitle`需要的配置设置

```jsx
var config = require('path/to/configuration');

var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
        />
      )
    }
  };

var OriginalTitle  = ({ title }) => <h1>{ title }</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

这个高阶组件，将 `OriginalTitle`组件需要接受一个`title`属性的细节，放在了高阶函数内部实现的，外部也不知道`OriginalTitle`组件的 `title`属性值来自 `config.appTitle` 对象中，这是一种代码隔离，有助于组件的测试，因为可以轻松创建模拟

这种模式的另外一个特点，是可以让我们有一个很好的缓冲区来添加新的逻辑，例如，如果将`OriginalTitle`组件的某个属性值需要从服务器中获取，那么就可以使用高阶函数，并且在生命周期中进行请求

```jsx

// 2. 定义一个函数，函数接受一个参数，返回一个 React 组件，并且在组件生命周期内进行请求，然后更新到 state里面
var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    constructor(props) {
      super(props);

      this.state = { remoteTitle: null };
    }
    componentDidMount() {
      fetchRemoteData('path/to/endpoint').then(data => {
        this.setState({ remoteTitle: data.title });
      });
    }
      // 下面进行渲染，当 state 被更新，组件就会重新渲染，这样就传递给了参数组件里面了
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
          remoteTitle={ this.state.remoteTitle }
        />
      )
    }
  };

// 1. 定义一个普通的函数组件
var OriginalTitle  = ({ title, remoteTitle }) =>
  <h1>{ title }{ remoteTitle }</h1>;

// 3. 调用高阶函数并将上面的组件传做参数
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

这样，`OriginalTitle` 组件将会直到自己有两个属性，但是不用关系数据从哪里来

## render props

而最近，React 社区开始流行一种新的方案， `render props`，在上面的例子中 `children`是一个 React 的组件，然而，`render props`中传递的是 `jsx表达式`，举个例子

```jsx
function UserName({ children }) {
  return (
    <div>
      <b>{ children.lastName }</b>,
      { children.firstName }
    </div>
  );
}

function App() {
  const user = {
    firstName: 'Krasimir',
    lastName: 'Tsonev'
  };
  return (
    <UserName>{ user }</UserName>
  );
}
```

这个有点意思，以前从来没这么写过，有点意思。

```jsx
// 声明一个组件并且遍历 todos 数组，调用 children 函数返回要渲染的 带b标签的 或者不带 标签的值
function TodoList({ todos, children }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ children(todo) }</li>
        ))
      }</ul>
    </section>
  );
}
// App组件
function App() {
  // 定义一个数组
    const todos = [
    { label: 'Write tests', status: 'done' },
    { label: 'Sent report', status: 'progress' },
    { label: 'Answer emails', status: 'done' }
  ];
    // 定义一个函数，返回布尔值
  const isCompleted = todo => todo.status === 'done';
  return (
      // 调用 TodoList 组件，并且传一个 函数作为 children
    <TodoList todos={ todos }>
      {
        todo => isCompleted(todo) ?
          <b>{ todo.label }</b> :
          todo.label
      }
    </TodoList>
  );
}
```

慢慢看一下还是可以看懂的，写这样的代码有点抽象了

```jsx
function TodoList({ todos, render }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ render(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

return (
  <TodoList
    todos={ todos }
    render={
      todo => isCompleted(todo) ?
        <b>{ todo.label }</b> : todo.label
    } />
);
```

这是另外一种，通过定义一个 render 属性，值是一个函数，并且在 TodoList 组件内部调用 render 传的函数实现渲染，这种模式比上面哪个可读性高多了

通过组合，我们可以做一些好玩的事情比如：

```jsx
// 定义一个组件
class DataProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = { data: null };
    // 上面定义 date 为 null ，下面定义一个定时器 五秒之后更新 state
      setTimeout(() => this.setState({ data: 'Hey there!' }), 5000);
  }
  render() {
      // 判断 date 是否为空，如果为空则不渲染
    if (this.state.data === null) return null;
    	// 不为空则调用 props 的 render 函数并且传入参数
      return (
      <section>{ this.props.render(this.state.data) }</section>
    );
  }
}
// 这里调用并传入 render 属性，值为一个函数接受一个参数 返回一个标签（这个参数好像没什么意义
<DataProvider render={ data => <p>The data is here!</p> } />
```

# 受控组件和非受控组件

受控组件和非受控组件常用语用户输入，例如 下面就是一个 受控组件

```jsx
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
    // 给input 组件传入 value 属性值为 state
  render() {
    return <input type='text' value={ this.state.value } />;
  }
};
```

上面代码确实是一个受控组件，但是用户无法更改`input`的值，因为的值指定的是一个固定的`state`这个`state`并不会更新永远都是`hello`，为了使值可以发生变化，需要添加`onChange`事件来更新和处理用户输入

```jsx
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return (
      <input
        type='text'
        value={ this.state.value }
        onChange={ this._change } />
    );
  }
  _handleInputChange = (e) => {
    this.setState({ value: e.target.value });
  }
};
```

这样添加了一个 `onChange`事件 并且指向 `_handleInputChange`函数，并且更新 state 刷新 UI 更新 input 的 value

##非受控组件

另一种是非受控组件，一般用于用户输入的值不重要不需要，只需要提供 `input`的 `defaultValue`属性即可，之后由浏览器保存用户输入值

```jsx
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return <input type='text' defaultValue={ this.state.value } />
  }
};
```

这样我们无法获取用户输入，这个组件的值更新没有更新我们都无法得知，也不知道用户输入了什么，为了直到用户输入了什么我们必须使用`refs`来获取真实`DOM`读取用户输入。

```jsx
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return (
      <input
        type='text'
        defaultValue={ this.state.value }
        onChange={ this._change }
        ref={ input => this.input = input }/>
    );
  }
  _handleInputChange = () => {
    this.setState({ value: this.input.value });
  }
};
```

这段代码很简单，就不做过多解释。尽量不要在 React 中使用非受控模式

# 展示组件和容器组件

展示组件和容器组件也被称为，木偶组件和智能组件，这些概念对初学者很不友好，在初学者比较常见的一个问题就是我的数据应该放在哪里，如果进行通信，而这个问题的答案往往是不同统一的，只有经过大量的练习拥有了充足的经验之后，才可能有答案，但是对于这种问题是有一个被广泛使用的模式的，有助于组织基于 React 的应用程序，那就是将组件分解为 `展示组件`和`容器组件`

从一个简单的例子开始说明，然后将组件分解为`展示组件`和`容器组件`

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: this.props.time };
  }
  render() {
    const time = this._formatTime(this.state.time);
    return (
      <h1>
        { time.hours } : { time.minutes } : { time.seconds }
      </h1>
    );
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _formatTime(time) {
    var [ hours, minutes, seconds ] = [
      time.getHours(),
      time.getMinutes(),
      time.getSeconds()
    ].map(num => num < 10 ? '0' + num : num);

    return { hours, minutes, seconds };
  }
  _updateTime = () => {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};

ReactDOM.render(<Clock time={ new Date() }/>, ...);
```

在例子中，显示出档期内的时间值，通过`setInterval`每秒更新状态，而且组件会被重新渲染，使它看起来更像一个时钟

## 问题

在这个组件中我们做了很多件事情，使它看起来很臃肿，不方便维护

- 组件会自己改变自身的状态，改变组件内部的 `time`可能并不是一个好的主意，因为只有计时器才知道当前值，如果系统的另一部分依赖于这些数据，则很难分享
- _formatTime 函数其实在做两件事，首先从日期中提取所需的信息，并且确保这些值始终以两位数字表示，如果它不提取函数的一部分，那么就没什么问题，但是正是由于它引用了另外的函数，那么他就会强行绑定在当前环境

### 提取容器组件

容器组件了解数据，知道数据来自哪里，知道业务逻辑的细节，接受信息并将其格式化，使其容易被展示组件使用，大佬们经常使用`class`来创建容器，因为可以提供一个缓冲区间，可以在其中插入自定义逻辑

```jsx
// Clock/index.js
import Clock from './Clock.jsx'; // <-- that's the presentational component

export default class ClockContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: props.time };
  }
  render() {
    return <Clock { ...this._extract(this.state.time) }/>;
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _extract(time) {
    return {
      hours: time.getHours(),
      minutes: time.getMinutes(),
      seconds: time.getSeconds()
    };
  }
  _updateTime = () => {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};
```

`ClockContainer`组件初始一个`state` 并且在 `componentDidMount`生命周期中调用`setInterval`每秒更新`state`，并且最后将毫秒数传给`Clock`组件

### 展示组件

展示组件只关心如何展现 UI 部分，这些组件不需要逻辑函数，也没有依赖，通常时限为`无状态组件`，表示他们内部不保留任何状态

```jsx
export default function Clock(props) {
  var [ hours, minutes, seconds ] = [
    props.hours,
    props.minutes,
    props.seconds
  ].map(num => num < 10 ? '0' + num : num);

  return <h1>{ hours } : { minutes } : { seconds }</h1>;
};
```

数据全靠`父组件传递`，本身只负责渲染 UI 部分

### 这样做的好处

这样做有什么好处，好处就是可以提高组件的可重用性，减少组件之间的耦合状态，将`逻辑`与`UI`分离开来，互相之间没有强耦合关系，逻辑不管UI，同理UI也不关心逻辑实现。

# 单向数据流

单向数据流是一种和 React 十分契合的一种模式，围绕这种模式，数据是单向的，即组件不会修改接收到的数据，只会监听这些数据变化，从而得到新的值，但是不会修改实际的数据，这个更新发生在另一个地方的另一个机制，并且该组件将会被重新渲染，并赋予新的值。

举个例子：

```jsx
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { flag: false };
    this._onButtonClick = e => this.setState({
      flag: !this.state.flag
    });
  }
  render() {
    return (
      <button onClick={ this._onButtonClick }>
        { this.state.flag ? 'lights on' : 'lights off' }
      </button>
    );
  }
};

// ... and we render it
function App() {
  return <Switcher />;
};
```

此时只有`Switcher`组件内部持有数据，也就是说`Switcher`是唯一知道`flag`的地方

那么接下来将这个 `flag`传递给别的对象：

```jsx
var Store = {
  _flag: false,
  set: value =>{
    this._flag = value;
  },
  get: () => {
    return this._flag;
  }
};

class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { flag: false };
    this._onButtonClick = e => {
      this.setState({ flag: !this.state.flag }, () => {
        this.props.onChange(this.state.flag);
      });
    }
  }
  render() {
    return (
      <button onClick={ this._onButtonClick }>
        { this.state.flag ? 'lights on' : 'lights off' }
      </button>
    );
  }
};

function App() {
  return <Switcher onChange={ Store.set } />;
};
```

这里通过`Switcher`的`button的onClic`事件触发修改flag，并且通过 `props`传下来的 `onChange`属性的值是一个函数，调用这个函数成功修改 `Store`对象的值。

思考一个问题，如果`Store`对象，被其他组件修改了，这里就发生了数据不同意 `Store`已经被更改了，`Switcher`组件内部的`flag`并没有更改，这就导致了数据不统一。

而单项数据流就是为了解决这个问题，他消除了状态存于多处，而导致的数据不统一，为了实现这一点，我们稍微对`Store`稍微进行一下调整

```jsx
var Store = {
    // 通知队列
  _handlers: [],
    // flag的值
  _flag: '',
    // 将通知对象添加进队列以便数据更改进行通知
  subscribe: handler => {
    this._handlers.push(handler);
  },
  set: value => {
    this._flag = value;
    this._handlers.forEach(handler => handler(value))
  },
  get: () => {
    return this._flag;
  }
};
```

这是一个简单的订阅通知模式，然后更新 App 组件，以便每次 Store 更改的时候重新渲染它：

```jsx
class App extends React.Component {
  constructor(props) {
    super(props);
	// 定义初始 state
    this.state = { value: Store.get() };
      // 将 App 的 setState 添加到带通知队列内Store._handlers 从 [] 变为了 [(value) => {this.setState({value})}]，等数据发生变化的时候执行队列函数，就可以更新 App 的 state 从而引发重新渲染
    Store.subscribe(value => this.setState({ value }));
  }
  render() {
    return (
      <div>
        <Switcher
          value={ this.state.value }
          onChange={ Store.set.bind(Store) } />
      </div>
    );
  }
};
```

由于存在了这种变化，`Switcher`变得非常简单，因此可以将`Switcher`写成一个无状态组件

```jsx
function Switcher({ value, onChange }) {
  return (
    <button onClick={ e => onChange(!value) }>
      { value ? 'lights on' : 'lights off' }
    </button>
  );
};

<Switcher
  value={ Store.get() }
  onChange={ Store.set } />
```

这种单向的数据模式，使得开发变得更加容易，无需担心数据的不统一，希望可以通过这个例子了解一下单向数据模式。

# Flux

作者迷恋于简化代码，但是并不是说越小越好，因为减少代码并不意味着更加易用，同时作者相信软件行业的很大一部分问题来自不必要的复杂性，复杂性是自己抽象的结果，程序员喜欢抽象，喜欢把东西放在黑盒内，并期望这些盒子能够一起工作，

Flux 是构建用户界面体系的设计模式，它是由 Facebook 在 F8 会议上退出的，从那之后，很多公司采纳了这个想法，这事故是构建前端应用程序的一个很好的模式，Flux 经常和 React 一起使用，这种模式简单而灵活，这种模式有助于更快的创建应用程序，同时保持代码组织良好

流程如下

Views ---> 触发 action ---> 通过 dispatch ---> 更新 Stores ---> 而Stores的更新会引起 View 重新渲染

这种模式中的最主要的部分是 dispatch ，它充当系统中所有实践的中心， 它的工作是接 action 并且传递给 Store，并通过内部状态/数据做出反应，这种变化触发了对 React 组件的重新渲染

这些 action 是从 View 或者其他部分进入 dispatch ，例如 HTTP 请求结束收到结果会触发一个 action ，说明请求成功。

