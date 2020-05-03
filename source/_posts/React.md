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
4. [Dan 讲解 Fiber](https://www.youtube.com/watch?v=aS41Y_eyNrU&feature=youtu.be)

# React API

[react/src/React.js](https://github.com/facebook/react/blob/master/packages/react/src/React.js)

React 本身只包含定义组件的核心 API ,它不包含调度算法以及其他平台特定代码.

## component

[React Component](https://github.com/facebook/react/blob/master/packages/react/src/ReactBaseClasses.js)

### React.Component

```js
// 帮助更新状态的基类组件
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject; //
  this.updater = updater || ReactNoopUpdateQueue;
}
```

基类储存了一些必要信息, 之后向原型添加了 isReactComponent 和 核心的 setState 方法

```js
Component.prototype.isReactComponent = {};
```

```js
/**
 * Sets a subset of the state. Always use this to mutate
 * state. You should treat `this.state` as immutable.
 * 设置一个状态的子集, 永远要使用他来改变 state, 并且你应该视为 `this.state` 是不变的
 *
 * There is no guarantee that `this.state` will be immediately updated, so
 * accessing `this.state` after calling this method may return the old value.
 *
 * 在这里不保证 `this.state` 会立即更新, 所以在调用此方法之后访问 `this.state` 会获得一个旧值
 *
 * There is no guarantee that calls to `setState` will run synchronously,
 * as they may eventually be batched together.  You can provide an optional
 * callback that will be executed when the call to setState is actually
 * completed.
 *
 * 在这里不保证 `setState` 是同步运行的, 他们可能是一起被批量处理的. 你可以提供一个回调函数,在 `setState` 执行完毕后会被运行.
 *
 * When a function is provided to setState, it will be called at some point in
 * the future (not synchronously). It will be called with the up to date
 * component arguments (state, props, context). These values can be different
 * from this.* because your function may be called after receiveProps but before
 * shouldComponentUpdate, and this new state, props, and context will not yet be
 * assigned to this.
 *
 * 如果给 setState 提供了一个函数, 它将会在未来被运行(不是同步), 它调用的时候会得到最新的(state, props, context)
 * 但是组件的其他 this.*(指 state, props, context) 可能不尽相同, 因为函数会在 receiveProps 之后
 * 但是在 shouldComponentUpdate 之前运行,这个时候,新的 state 还没有合并到 this
 *
 *
 * @param {object|function} partialState Next partial state or function to
 *        produce next partial state to be merged with current state.
 *  下一个局部的 state 或者通过函数生成 新的 state, 值将会被合并到当前的 state内
 * @param {?function} callback Called after state is updated.  回调函数
 * @final
 * @protected
 */
Component.prototype.setState = function (partialState, callback) {
  // 参数检查
  invariant(
    typeof partialState === "object" ||
      typeof partialState === "function" ||
      partialState == null,
    "setState(...): takes an object of state variables to update or a " +
      "function which returns an object of state variables."
  );
  // this.updater 是实际渲染器传入的,并非 React 核心提供.
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};

/*
 * @param {?function} callback Called after update is complete.
 * @final
 * @protected
 */
// forceUpdate 和 setState 其中一个比较大的差异是 forceUpdate 不会触发 `shouldComponentUpdate`
Component.prototype.forceUpdate = function (callback) {
  this.updater.enqueueForceUpdate(this, callback, "forceUpdate"); // 这里第三个参数不同
};
```

### React.PureComponent 

```js
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

function PureComponent(props, context, updater) {
  //...和 Component 一样
}
// 这部分 关于原型 继承 如果看不懂可以看这个教程 https://zh.javascript.info/prototypes
const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;
// Avoid an extra prototype jump for these methods.
Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true; // 多了一个 flag
```

### React.memo

## create react element

### React.createElement

### React.createFactroy

## transition element

### React.cloneElement

### React.isValidElement

### React.Children

## Fragments

### React.Fragment

## Refs

### React.createRef

### React.forwardRef

## Suspense

### React.lazy

### React.Suspense

## Hook

base hooks

### useState

### useEffect

### useContext

additional Hooks

### useReducer

### useCallback

### useMemo

### useRef

### useImperativeHandle

### useLayoutEffect

### useLayoutEffect

### useDebugValue
