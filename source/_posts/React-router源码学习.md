---
title: React-router源码分析
date: 2019-05-11 13:53:25
tags: React
---

React-router 源码学习

<!-- more -->

# 开篇

开篇通过一个最简单的 React-Router 的 demo 来进行学习

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

function Index() {
  return <h2>Home</h2>;
}

function About() {
  return <h2>About</h2>;
}

function Users() {
  return <h2>Users</h2>;
}

function AppRouter() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about/">About</Link>
            </li>
            <li>
              <Link to="/users/">Users</Link>
            </li>
          </ul>
        </nav>

        <Route path="/" exact component={Index} />
        <Route path="/about/" component={About} />
        <Route path="/users/" component={Users} />
      </div>
    </Router>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<AppRouter />, rootElement);
```

这个来自官网的 demo,其中有四个组件分别讲解一下

1. `BrowserRouter`
2. `Router`
3. `Route`
4. `Link`

# BrowserRouter

BrowserRouter 作为最外层的组件,它需要是所有的 Route 的父组件.

BrowserRouter 源码比较短下面直接展示一下

```jsx
import React from "react";
import { Router } from "react-router";
import { createBrowserHistory as createHistory } from "history";
import PropTypes from "prop-types";
import warning from "tiny-warning";

/**
 * The public API for a <Router> that uses HTML5 history.
 * 公开一个基于 HTML5 history 的 api <Router>
 */
class BrowserRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}

if (__DEV__) {
  BrowserRouter.propTypes = {
    basename: PropTypes.string,
    children: PropTypes.node,
    forceRefresh: PropTypes.bool,
    getUserConfirmation: PropTypes.func,
    keyLength: PropTypes.number
  };

  BrowserRouter.prototype.componentDidMount = function() {
    warning(
      !this.props.history,
      "<BrowserRouter> ignores the history prop. To use a custom history, " +
        "use `import { Router }` instead of `import { BrowserRouter as Router }`."
    );
  };
}

export default BrowserRouter;
```

这个组件,非常简单,通过引入和初始化 `history` 包来获得一个 history 对象, 然后在 componentDidMount 生命周期中添加了一个检查, 这个检查是判断 props 中是否有 history 属性,如果存在的话,那么就弹出一个警告, 如果要使用自定义 history 对象那么就需要使用 import {Router} from 'react-router-dom'来使用. 最后这个组件返回了 `react-router` 包中的 `Router` 组件.

# Router

Router 组件篇幅比较长, 我会忽略一些样板代码,通常是一些 import 和 if(**DEV**), **DEV** 是作者设定的一个全局变量,这个变量等于 `'production' === process.env.NODE_ENV` 是用来判断当前是否处于开发环境.

```jsx

// Router 组件中使用的 context 并不是 React自带的,因为React16版本的context不支持React15,因为React-router需要兼容React15所以使用了一个第三方实现的context,而这个第三方实现的context使用的是React15 的旧版本 context.
import RouterContext from "./RouterContext";
/**
 * The public API for putting history on context.
 * 通过 context 向下传播 history
 */
class Router extends React.Component {
// 这个静态方法用用户生成 match对象
  static static computeRootMatch(pathname) {
    return { path: "/", url: "/", params: {}, isExact: pathname === "/" };
  }

  constructor(props) {
    super(props);

    // 这里使用state储存props传递下来的history的location(主要是用于通过setState更改location来触发重新渲染,因为BrowserRouter仅仅是初始化和传递了history对象)
    this.state = {
      location: props.history.location
    };

    // This is a bit of a hack. We have to start listening for location
    // changes here in the constructor in case there are any <Redirect>s
    // on the initial render. If there are, they will replace/push when
    // they mount and since cDM fires in children before parents, we may
    // get a new location before the <Router> is mounted.

    // 开始通过history对象提供的监听方法,添加监听事件,
    // 如果有在构造函数还在执行的时候location发生了任何变化,需要记录下来.

    this._isMounted = false; // 用于监听组件是否加载,这个变量会在didMount周期内赋值为true
    this._pendingLocation = null; // 用于储存最新的location

    if (!props.staticContext) { // 如果 staticContext 对象不存在,那么执行方法 staticContext 对象是在服务端渲染存在且必须的
      this.unlisten = props.history.listen(location => { // 添加监听事件
        if (this._isMounted) { // 如果组件已经加载,不过理论上还在执行构造函数组件不肯能已经加载
          this.setState({ location });
        } else {// 否则使用变量来记录location
          this._pendingLocation = location;
        }
      });
    }
  }

  componentDidMount() {
    this._isMounted = true; // 组件已经加载 赋值为 true

    if (this._pendingLocation) {// 如果不是服务端渲染那么_pendingLocation是存在值得更新到state内
      this.setState({ location: this._pendingLocation });
    }
  }

  componentWillUnmount() { // 清理工作处理
    if (this.unlisten) this.unlisten();
  }

  render() {
    return ( // 返回一个 context 组件
      <RouterContext.Provider
        children={this.props.children || null} // 传递children
        value={{ // 传递 context 的值
          history: this.props.history, // history
          location: this.state.location,// location
          match: Router.computeRootMatch(this.state.location.pathname), // match对象
          staticContext: this.props.staticContext // 这个服务端渲染才会存在
        }}
      />
    );
  }
}

  Router.prototype.componentDidUpdate = function(prevProps) { // 检查
    warning(
      prevProps.history === this.props.history,
      "You cannot change <Router history>"
    );
  };
}

export default Router;
```

# Switch

这里先介绍一个 Switch 组件, 看过文档的应该知道 Switch 组件的作用是只匹配一个完全符合 location 的 Router 组件,怎么理解呢举个例子

```jsx
function AppRouter() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about/">About</Link>
            </li>
            <li>
              <Link to="/users/">Users</Link>
            </li>
          </ul>
        </nav>

        <Route path="/" component={Index} />
        <Route path="/users" component={Users} />
        <Route path="/about/" component={About} />
      </div>
    </Router>
  );
}
```

如上组件如果不带没有 switch 组件包裹那么当导航到`/about` 和 `/users`'的时候其他两个也会被渲染,因为`/about` 和 `/users` 中都包含了 `/`

那么加上`Switch`之后就会避免这种情况,会进行精准匹配.

```jsx
<Switch>
  <Route path="/" component={Index} />
  <Route path="/users" component={Users} />
  <Route path="/about/" component={About} />
</Switch>
```

源码如下

```jsx
import RouterContext from "./RouterContext";
import matchPath from "./matchPath";

/**
 * The public API for rendering the first <Route> that matches.
 * Switch 组件渲染第一个符合条件的 Route 组件
 */
class Switch extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {" "}
        // 订阅 context
        {context => {
          invariant(context, "You should not use <Switch> outside a <Router>"); // 检查context是否存在

          const location = this.props.location || context.location;

          let element, match;

          // We use React.Children.forEach instead of React.Children.toArray().find()
          // here because toArray adds keys to all child elements and we do not want
          // to trigger an unmount/remount for two <Route>s that render the same
          // component at different URLs.
          // 通过React 提供的children的方法遍历children,
          React.Children.forEach(this.props.children, child => {
            if (match == null && React.isValidElement(child)) {
              // 如果 match 为空而且 child是react组件
              element = child;

              const path = child.props.path || child.props.from; // Router的path参数

              match = path
                ? matchPath(location.pathname, { ...child.props, path }) // 解析参数和生成match对象
                : context.match;
            }
          });

          return match
            ? React.cloneElement(element, { location, computedMatch: match }) // 返回一个克隆的组件并添加location和 computedMatch 对象
            : null;
        }}
      </RouterContext.Consumer>
    );
  }
}
```

# Route

Route 是 BrowserRouter 或者 Switch 的子组件,主要是根据 path 和 location 进行匹配渲染.
代码删掉了警告和开发验证.

```jsx
import RouterContext from "./RouterContext";
import matchPath from "./matchPath";

function isEmptyChildren(children) {
  return React.Children.count(children) === 0;
}

/**
 * The public API for matching a single path and rendering.
 * Route 组件, 根据path进行渲染
 */
class Route extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {" "}
        // context
        {context => {
          invariant(context, "You should not use <Route> outside a <Router>"); // 判断顶级是否存在 Router 组件,如果不存在进行报错,

          const location = this.props.location || context.location; //获得location
          const match = this.props.computedMatch // 判断 是否存在 computedMatch 对象, computedMatch对象是Switch组件传递下来的, 如果存在就使用如果不存在那么自己解析
            ? this.props.computedMatch // <Switch> already computed the match for us
            : this.props.path
            ? matchPath(location.pathname, this.props)
            : context.match;

          const props = { ...context, location, match };

          let { children, component, render } = this.props;

          // Preact uses an empty array as children by
          // default, so use null if that's the case.
          if (Array.isArray(children) && children.length === 0) {
            // 判断children是否为一个数组 ,api要求是一个函数
            children = null;
          }

          if (typeof children === "function") {
            // 判断children是否是一个函数
            children = children(props); //如果是就将props传递给它 然后由children函数自己决定渲染

            if (children === undefined) {
              // 如果children不存在

              children = null;
            }
          }

          return (
            <RouterContext.Provider value={props}>
              {children && !isEmptyChildren(children) //如果children属性存在,且是空的
                ? children // 返回空的children
                : props.match // 判断match 是否匹配
                ? component // 如果 component 存在
                  ? React.createElement(component, props) // 渲染component
                  : render // 如果component不存在render存在
                  ? render(props) // 调用render
                  : null // 不存在返回空
                : null}{" "}
              // match不匹配返回空
            </RouterContext.Provider>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}
```

Route 主要做几件事情, 从顶级的 Router 组件中接受 location 属性 (也可以是 Route 组件的 location 属性), 判断用户是使用 children 还是 component 还是 render 进行渲染, 根据 location 进行匹配渲染,如果符合就返回,如果不符合就返回 null, 如果 Switch 存在那么 Switch 会传递一个 computedMatch 属性, Route 则会根据它进行渲染.

# Link

```jsx
function isModifiedEvent(event) {
  return !!(event.metaKey || event.altKey || event.ctrlKey || event.shiftKey);
}

/**
 * The public API for rendering a history-aware <a>.
 */
class Link extends React.Component {
  handleClick(event, history) {
    try {
      if (this.props.onClick) this.props.onClick(event);
    } catch (ex) {
      event.preventDefault();
      throw ex;
    }

    if (
      !event.defaultPrevented && // onClick prevented default
      event.button === 0 && // ignore everything but left clicks
      (!this.props.target || this.props.target === "_self") && // let browser handle "target=_blank" etc.
      !isModifiedEvent(event) // ignore clicks with modifier keys
    ) {
      event.preventDefault();

      const method = this.props.replace ? history.replace : history.push;

      method(this.props.to);
    }
  }

  render() {
    const { innerRef, replace, to, ...rest } = this.props; // eslint-disable-line no-unused-vars

    return (
      <RouterContext.Consumer>
        {" "}
        // 从context中读取context
        {context => {
          invariant(context, "You should not use <Link> outside a <Router>"); // 如果不存在则报错

          const location =
            typeof to === "string"
              ? createLocation(to, null, null, context.location)
              : to;
          const href = location ? context.history.createHref(location) : ""; // 调用函数,根据模式生成对应的url

          return (
            <a
              {...rest}
              onClick={event => this.handleClick(event, context.history)}
              href={href}
              ref={innerRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  }
}
```

Like 看代码就非常简单了, 它主要会根据 Router 的 basename 属性(如果有), 和当前模式(如果是 hash 模式那么会加上#号), 然后处理 Click 事件.

# withRouter

withRouter 组件主要是给任意组件添加 `history` `location` `match` `staticContext(服务端渲染属性)`,其核心就是通过 context 得到上述属性,然后传递给指定的组件.

代码没什么好讲解的就是一个 HOC 高阶组件.

```jsx
/**
 * A public higher-order component to access the imperative API
 */
function withRouter(Component) {
  const displayName = `withRouter(${Component.displayName || Component.name})`;
  const C = props => {
    const { wrappedComponentRef, ...remainingProps } = props;

    return (
      <RouterContext.Consumer>
        {context => {
          invariant(
            context,
            `You should not use <${displayName} /> outside a <Router>`
          );
          return (
            <Component
              {...remainingProps}
              {...context}
              ref={wrappedComponentRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  };

  C.displayName = displayName;
  C.WrappedComponent = Component;

  if (__DEV__) {
    C.propTypes = {
      wrappedComponentRef: PropTypes.func
    };
  }

  return hoistStatics(C, Component);
}
```

# 常见疑问

## BrowserRouter 和 HashRouter 的区别

BrowserRouter 使用 HTML5 的 history 属性来进行管理,[兼容查询](https://caniuse.com/#search=history)
HashRouter 使用锚点来进行管理,浏览器在 hash 发生变化的时候回触发事件,从而触发重新渲染,用于兼容低版本浏览器.

上述两个还有两个组件还有一个区别,就是 BrowserRouter 使用的 HTML5 history 产生的 url 是一个真实的 url, 每个地址都会向后端发送一个请求,那么需要后端正确的处理这个请求的响应,否则就会 404,至于后端如何解决这个问题,最简单的就是让后端在 NGINX 配置里面将所有的访问都响应返回`index.html`

而 HashRouter 实际上是使用的锚点 hash 值作为记录,不管 hash 值如何发生变化实际上 url 都没有发生变化,不会向后端发送请求,也不需要后端进行处理.

## React-router-dom 和 React-router 的区别和联系

先说区别`React-router-dom` 提供了 `BrowserRouter` `HashRouter` `Link` `NavLink` 四个独有的组件,这四个组件是 React-router 中不存在的,也就是说你要使用这四个 api 你就必须安装和使用 React-router-dom.

他们两者有什么联系, 通过分析源码发现 `BrowserRouter` `HashRouter` 两个组件实际上都是调用的 React-router 的 `Router` 组件, 但是 `BrowserRouter` `HashRouter` 组件在内部调用了 history 包,添加了监听事件,生成了 history 属性.

简单了说 React-router 是一个更为基础的包, React-router-dom 基于它提供了一些组件和方法,可以直接就用.
