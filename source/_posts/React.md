---
title: React
date: 2020-03-23 16:44:35
tags: React
---

React 学习

<!-- more -->

参考资料 :

1. [React 代码库组成](https://reactjs.org/docs/codebase-overview.html)
2. [React core API 设计简介](https://reactjs.org/docs/implementation-notes.html)
3. [从零开始构建 React 2016](https://www.youtube.com/watch?v=_MAD4Oly9yg)

# React API

[react/src/React.js](https://github.com/facebook/react/blob/master/packages/react/src/React.js)

入口文件,导出了对外使用的所有格 API, 例如 createElement , cloneElement 等.

## jsx 和 createElement

在编写 React 的时候通常会直接使用

```jsx
<div>hello</div>
```

这种语法来编写代码, 这种语法被称为 `jsx` ,而它实际上是 `React.createElement` 的语法糖,这也就是为什么写函数组件也必须要求 import React 的原因, 这个背后的转换工作由 babel 的相关插件完成.

```jsx
//jsx

<div id="divId" classNmae="divClass">
  <span>hello</span>
  <span>world</span>
</div>;

//js

("use strict");

/*#__PURE__*/
React.createElement(
  "div",
  {
    id: "divId",
    classNmae: "divClass"
  },
  /*#__PURE__*/ React.createElement("span", null, "hello"),
  /*#__PURE__*/ React.createElement("span", null, "world")
);

// 备注 /*#__PURE__*/ 是babel编译时的优化标注,表示当前函数是纯函数,用于代码压缩插件识别.
```

[react/src/](https://github.com/facebook/react/blob/master/packages/react/src/ReactElement.js)

函数签名

```tsx
function createElement(type: string | function | React.component, config: {k:string: any}, children: ReactNode){}
```

type 参数可以为 string(对应 html tag) 或 function (对应 function component) 或者 class (对应 class component)

```jsx
//jsx
<div />;

//js
React.createElement("div", null);

// jsx

function Div() {
  return null;
}

// js
React.createElement(Div, null);
```

可以看到差异就是对于小写字母 type 都是 string,而对于大写字母开头的都传递的是引用.

之后 createElement 函数会对参数进行预处理, 例如收集关键 props key (ref,key 等关键 props),判断 children 的数量, 并赋值给 props.children

之后会调用 ReactElement 并传入处理完成的参数.

```jsx
return ReactElement(
  type,
  key,
  ref,
  self,
  source, // 编译工具会传入函数位置信息,用于显示报错.
  ReactCurrentOwner.current, // 负责记录创建此元素的组件
  props
);
```

而 ReactElement 函数会返回一个对象

```jsx
{
    $$typeof: REACT_ELEMENT_TYPE,  // 一个 Symbol 对象 用来确定某个对象是否为 React element
    type: type, // 组件实际类型
    key: key, // key
    ref: ref,
    props: props,
    _owner: owner,// 负责记录创建此元素的组件
}
```

## React.Component and React.PureComponent

[react/ReactBaseClasses.js](https://github.com/facebook/react/blob/master/packages/react/src/ReactBaseClasses.js)

```jsx
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}
```

这是一个普通的构造函数(es5 的类) 接收仨参数,前两个明确,第三个暂不明确

```jsx
Component.prototype.isReactComponent = {};
// 用于标记当前组件是class 组件,因为 class 组件需要 new 而函数组件不需要也不能 new
```

- setState

setState 函数内部调用了 this.updater.enqueueSetState() 方法, updater 还不明确

- forceUpdate

forceUpdate 函数内部直接调用了 this.updater.enqueueForceUpdate() 同上, updater 还不明确

PureComponent 在 prototype 原型链中多了一个属性 `isPureReactComponent` 值为 true, 表明这是一个 PureComponent
