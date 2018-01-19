---
title: Router4
date: 2017-11-27 19:57:58
tags: Router
categories: 
	- 教程
---
### Router4
## 参考至[精读Router4](https://zhuanlan.zhihu.com/p/31178105)
> 入门学习的时候，本着学新不学旧的原则，啃了Router4，这里感谢大佬将官网翻译了一下。(有幸还和参与翻译的大佬聊了两句。)

#### 代码分割
在Router3中有函数钩子，可以做到代码分割，而Router4中的Router本身只是一个普通组件，没有钩子，这里我们用到`react-loadable`，可以做到路由甚至组件化的动态加载(按需加载)。

> npm i react-loadable
```jsx
import Loadable from 'react-loadable'

// 新建一个组件,这里用Loadable包装一下Home组件
const AsyncHome = Loadable({
    loader: () => import('./Home'),
    loading: () => null
})
export default AsyncHome

```
[import()介绍](https://webpack.js.org/api/module-methods/#import-)

Loadable接收一个对象作为参数，第一个参数是loader要按需加载的组件，第二个是loadeing 过场动画，上面的例子上是返回一个空值。然后包装后的组件，就可以这样用
```jsx
import AsyncHome from './AsyncHome'

<Route path='/' component={AsyncHome} />

```
#### 转场动画
这个我没用过用过之后回来补上[AntM](https://motion.ant.design/)

#### 滚动条复位

```jsx
<Router history={history}>
  <ScrollToTop>
    <div>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route path="*" component={NotFound} />
      </Switch>
    </div>
  </ScrollToTop>
</Router>

class ScrollToTop extends Component {
    componentDidUpdate(prevProps) {
        if (this.props.location !== prevProps.location) {
            window.scrollTo(0, 0)
        }
    }

    render() {
        return this.props.children
    }
}
export default withRouter(ScrollToTop)
```
这段代码，应该挺好理解的，我没有这么用过，不过后面一定会用。

#### 嵌套
简单的看了一点Router3，发现3和4的嵌套是大不相同的
3里面你可以这样写
```jsx
<Route path='/' component={App}>
    <Route path='/hoem' component={Home} />
</Route>
```
Route标签套Route标签，对于一些人还是很好理解的，ps我当初学的时候也深受其毒害（看不懂4的文档，借鉴了一下3然后就。。）

前面说了Router4本质上就是普通的React组件所以Router4应该这样写

```jsx
Router.js
<Router>
    <Route path='/' component={App}
</Router>

App.js
<App>
<Route path='/home' component={Home}
</App>
```

这样把Route写到组件里面，当成普通的组件使用，但是有人不喜欢这样。那好吧，还有另一种方案。

```jsx
<Router>
    <Switch>
        <Route path='/' component={App} />
        <Route path='/home' render={props => <App><Home {...props} /></App>}
    <Switch>
</Router>
```

这里Switch标签的目的是只匹配一个路由，如果不加的话访问/会渲染两个，加了只会渲染App路由，第二个就是render，render接受一个函数，函数会被附加一个参数，参数是Router的所有方法和信息，所以我们把这个附加给Home组件传给他。

Router的render还可以用来处理一些逻辑，比如函数里面可以判断一些东西，然后选择重定向或者渲染组件。

#### 服务端渲染

这部分我还不会，有兴趣的大佬可以看原文章，等我学会了我再补充哈。





