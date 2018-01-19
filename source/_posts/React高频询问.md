---
title: React高频询问
date: 2017-11-17 19:51:0
tags:  解答
categories: 
	- 教程
---
##  前言
参考掘金文章[Vue脱坑记](https://juejin.im/post/59fa9257f265da43062a1b0e), 感觉很不错，参考大佬文章整理一下React询问比较多的问题。
###  npm下载很慢或者下载失败
此问题比较困扰新手，npm由node一同发布，但由于众所周知的原因导致下载速度过慢，甚至无法下载情况解决方案有以下两种：
1. NPM换源
故名思议，就是把NPM的下载服务器更换为国内的服务器，国内当然是大厂阿里了。

>  npm config set registry https://registry.npm.taobao.org   (npm config get registry 验证是否更换成功）
> yarn config set registry https://registry.npm.taobao.org    (yarn为Facebook出的包管理工具）

2. 使用淘宝出的NPM替代品`CNPM`  (CNPM是使用文件夹快捷方式使用的，有时候会因为这个出现莫名BUG所以还是推荐第一种方法）

>  npm install -g cnpm ---registry=https://registry.npm.taobao.org
>  安装成功之后所有需要使用NPM的地方换成CNPM即可 比如NPM i React 就可以写成CNPM i React

###  NPM安装NODE-SASS 使用SASS但是无法下载NODE-SASS
因为NODE—SASS在GitHub托管的是源码，需要用户自己编译。而编译需要Python和Visual C ++构建环境（Windows同学，其他操作系统Linux以及Mac不需要）最简单的办法就是通过上面的办法进行换源，淘宝源上的是编译好的，如果非要自己编译，请自行安装Python2.7以及[Visual C ++生成工具](http://landinghub.visualstudio.com/visual-cpp-build-tools),具体请参考[官方文档](https://github.com/nodejs/node-gyp#on-windows)
`更新` NODE-SASS 只有CNPM 下载下来的是编译好的

###  `can't not find 'xxModule'` 找不到相关资源
出现这个问题有两种原因：
1. 引用路径有问题，npm包是直接Import React from ‘react’ 只需要写包名即可。而引用组件，静态资源是需要写相对路径的（有个问题就是所有文件里面的静态资源都需要import引入或者require引入，否则webpack无法得知依赖关系，而导致找不到）。
2. 第二种就是相关包没有下载或者下载不齐全，项目根目录重新命令行NPM i 即可

###  React的给return出来的HTML添加事件报错this.xxxx.xxxx not a function
在class和模块的内部。默认是使用严格模式的，以及React的基类 `Component` 中所有的this都是指向class本身的，但是return出来的HTML 会被转化成实际的dom这个时候的this指向就不是定义的时候的那个class（个人肤浅理解)举例说明
```jsx
import React, { Component } from  'react'
class  APP  extends  Component {
OnClick () {
  console.log(2333)
}
render () {
  return (
  	<button  onClick={this.OnClick}>按钮</button>
	)
  }
}
export default APP
```
以上代码逻辑上点击按钮会触发OnClick函数，但是实际上只会报错。
解决方法(推荐度由高至低)：
1.使用提案阶段语法静态属性的[提案](https://github.com/tc39/proposal-class-fields)，改提案在写累的实例属性的时候可以使用等式，从而将属性和方法写入类中。（提案阶段语法，需要单独的[babel插件](http://babeljs.io/docs/plugins/transform-class-properties/)  官方脚手架create-react-app 已有此项配置(需要使用antd，sass，代码检查功能的可以参考这个[脚手架](https://gitee.com/HiMrHu/ReactGuanFangJiaoShouJia)
```jsx
import React, { Component } from  'react'
class  APP  extends  Component {
OnClick  = () => {
	console.log(2333)
	}
render () {
return (
	<button  onClick={this.OnClick}>按钮</button>
   )
  }
}
export default APP
```
2.在构造函数中bind(this)
```jsx
import React, { Component } from  'react'
class  APP  extends  Component {
constructor () {
	super()
	this.OnClick = this.OnClick.bind(this)
}
OnClick () {
  console.log(2333)
}
render () {
  return (
  	<button  onClick={this.OnClick}>按钮</button>
	)
  }
}
export default APP
```
3.使用匿名箭头函数返回方法
此方法会生成不可服用的匿名函数，但是这个方法是可以主动给调用方法传参数的，慎重使用。
```jsx
import React, { Component } from  'react'
class  APP  extends  Component {
OnClick () {
  console.log(2333)
}
render () {
  return (
  	<button  onClick={ () => this.OnClick }>按钮</button>
	)
  }
}
export default APP
```
4.还有一种方法就是在jsx中bind，这个方法也可以给方法传参数而且不会产生不可复用的匿名函数，但是不好维护。和方法3各有优缺点吧（代码检查不允许在jsx中bind，所以放在了最后）
```jsx
import React, { Component } from  'react'
class  APP  extends  Component {
OnClick () {
  console.log(2333)
}
render () {
  return (
  	<button  onClick={ this.OnClick.bind(this) }>按钮</button>
	)
  }
}
export default APP
```
### 使用Fetch，但是ie浏览器报错Fetch未定义，及其他浏览器set未定义Map未定义
Fetch是浏览器原生支持的，但是ie浏览器全系列都不支持promise，解决方案
```
npm install whatwg-fetch
```
然后项目`入口`文件内

```
import 'whatwg-fetch'
```
React使用了Es6的Set和Map数据结构低版本浏览器不支持，一劳永逸型方案(此方案是吧所有es6特性都通过es5函数模拟，弊端是会增加js文件体积，也可以自己百度挑单独的polyfill添加）
```
npm install --save babel-polyfill
入口文件内
import 'babel-polyfill'
```
###  跨域问题
这个问题我也快被问烂了，真的太多人遇到了都不知道怎么吐槽
	1.使用`CORS` 跨域方案，后端配置前端无忧请求只支持IE10+。
	2.使用`nginx`反向代理，一劳永逸。
	3.Webpack有开发代理功能，自己去Webpack官网看，我记不住
	4.React官方脚手架`create-react-app`看[这里](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#proxying-api-requests-in-development)

###  为什么我用import style from ‘../../xxx.css’不管用啊还有为什么要这么用啊

这种是为了使用css模块化，需要在webpack中配置看[这里](https://doc.webpack-china.org/loaders/css-loader/)
其原理是给你的css类名随机化，或者添加随机后缀达到全局不会重复。用不用都行,自己衡量。

###  为什么我使用BrowserRouter开发环境正常，部署之后刷新就是404
这个问题是因为BrowserRouter 使用了HTML5的api导致每次url变化都是实际向服务器进行请求，所以需要后端收到前端路由请求之后都返回index.html。实在不会弄就去用HashRouter，url变化不会产生额外请求，因为他本质上一直是在/路径下

### 我想拦截页面。或者在页面进入之前做一些事情
请使用:
```js
<Route path="/home" render={(props) => <div {...props}>Home</div>}/>
```
render接受一个函数会向函数附加一个参数，这个参数是所有的Router方法和参数，记得传给组件，可以在return之前做一些判断。

###   ` xxx is not a function`  这个问题我被问了无数遍
我不知道怎么解释，我也不想解释。

###   `Cannot read property 'xxx' of undefined"` 这个问题比上面那个问题问的少一点或许吧。
这个一般都是组件内部引用了props传下来的值，但是父组件没传，或者引用了不存在的值。自己仔细检查。

###   `token: operator xxxxx` 这个问题问的人少。
少了括号，多了逗号，多了括号巴拉巴拉的。

###   `npm run build`之后我打开index.html不能访问啊
	1. 给后端，让他给你弄。
	2. 参考上一条。
	3. 还不死心？那就打开index.html 吧文件路径改成./开头的就好了。

###   `axios`的 `post` 请求后台接受不到 (直接抄原文的，我没遇到过，自己弄得垃圾node也只接受json格式化）

`axios`默认是 json 格式提交,确认后台是否做了对应的支持;

若是只能接受传统的表单序列化,就需要自己写一个转义的方法...

当然还有一个更加省事的方案,装一个小模块`qs`
```js
npm install qs -S 
// 然后在对应的地方转就行了..单一请求也行,拦截器也行...我是写在拦截器的. 
// 具体可以看看我 axios 封装那篇文章 //POST传参序列化(添加请求拦截器) Axios.interceptors.request.use( config => { 
// 在发送请求之前做某件事 
if ( config.method === "post" ) { 
// 序列化 
config.data = qs.stringify(config.data); 
// ***** 这里转义 } 
// 若是有做鉴权token , 就给头部带上token 
if (localStorage.token) { config.headers.Authorization = localStorage.token; } return config; }, error => { Message({ 
// 饿了么的消息弹窗组件,类似toast showClose: true, message: error, type: "error.data.error.message" }); 
return Promise.reject(error.data.error.message); } );
```
###   `[...Array]`,`...mapState`,`[SOME_MUTATION] (state) {}`,`increment ({ commit }) {}`是什么
数组解构,对象解构,对象风格函数,对象解构赋值传递（第三个我也不知道）

###  redux 的用户信息为什么还要存一遍在浏览器里(sessionStorage or localStorage)
因为 `Redux`的 store 干不过刷新啊.
保存在浏览器的缓存内,若用户刷新的话,在从里面取一下信息

###  有什么React+Redux+Router学习项目吗

偷偷放上自己的[代码](https://github.com/HiMrHu/cnode)

### nginx部署
传送门:[一篇不大靠谱的nginx 1.11.10配置文件](https://link.juejin.im/?target=https%3A%2F%2Fjuejin.im%2Fpost%2F58bfc412da2f60124db5999a)

### 为什么我的 `npm` 或者 `yarn` 安装依赖会生成 `lock`文件,有什么用
版本锁工具，以前有次npm出现了npm到的包不是自己想要的，所以就用lock这个文件记录了下载地址包版本，加密的值。用于控制版本。保证包的前后一致。

###  `package.json`里面的`dependencies` 和`devDependencies`的差异
其实不严格的话,没有特别的差异;
若是严格,遵循官方的理解;

*   `dependencies` : 存放线上或者业务能访问的核心代码模块,比如 `vue`,`vue-router`;
*   `devDependencies`: 处于开发模式下所依赖的开发模块,也许只是用来解析代码,转义代码,但是不产生额外的代码到生产环境, 比如什么`babel-core`这些

新版的npm已经不需要`--save` `--save-dev` 这指令的，当然也可以加上，npm会根据包的说明摆放位置的。
再说一遍代码打包看的是引入以及依赖关系，和在`dependencies`还是`devDependencies`放的没有一毛钱关系！！！！！！！！！！


