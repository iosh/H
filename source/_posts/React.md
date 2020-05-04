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

```js
export function memo<Props>(
  type: React$ElementType,
  compare?: (oldProps: Props, newProps: Props) => boolean
) {
  const elementType = {
    $$typeof: REACT_MEMO_TYPE,
    type,
    compare: compare === undefined ? null : compare,
  };
  return elementType;
}
```

构造了一个 ReactElement 对象, 通过 \$\$typeof 指明这是一个 react memo type

## create react element

[React.createElement](https://github.com/facebook/react/blob/master/packages/react/src/ReactElement.js)

### React.createElement

在 React 中使用 jsx 语法来编写代码,但是实际上 jsx 是 React.create.Element 的一个语法糖

```jsx
<div class="root">hello</div>
```

上面的代码会被 babel 工具(配合对应插件)转化为如下形式

```js
React.createElement(
  "div",
  {
    class: "root",
  },
  "hello"
);
// 形式就是
// function createElement(type, props, children)
```

```js
export function createElement(type, config, children) {
  // 省略一些信息,主要是 type 判断, config 检查 和 children 数量检查

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props
  );
}
const ReactElement = function (type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    // 这个标签永远使用一个独特的标识,来表名这是一个React Element
    // react 所有标识详见 packages/shared/ReactSymbols.js
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  // 省略一些检查和开发帮助

  return element;
};
```

### React.createFactory

React.createFactory 使用比较少,使用方法如下

```jsx
function App() {
  const div = React.createFactory("div");
  return div({ className: "app" }, "hello world");
}
```

源代码很简单

```js
export function createFactory(type) {
  const factory = createElement.bind(null, type);
  factory.type = type;
  return factory;
}
```

## transition element

主要是一些 helper api 帮助实现一些功能

### React.cloneElement

```jsx
React.cloneElement(
  React.createElement("div", null, "hello world"),
  newProps,
  newChildren
);
```

核心就是对一个 React element 元素进行 clone

```js
export function cloneElement(element, config, children) {
  const props = Object.assign({}, element.props); // 复制原有props

  // 复制其他属性
  let key = element.key;
  let ref = element.ref;
  const self = element._self;
  const source = element._source;
  let owner = element._owner;
}
//...
// 属性覆盖,如果有新的 props 那就覆盖原有元素的
if (config != null) {
  if (hasValidRef(config)) {
    ref = config.ref;
    owner = ReactCurrentOwner.current;
  }
  if (hasValidKey(config)) {
    key = "" + config.key;
  }

  let defaultProps;
  if (element.type && element.type.defaultProps) {
    defaultProps = element.type.defaultProps;
  }
  for (propName in config) {
    if (
      hasOwnProperty.call(config, propName) &&
      !RESERVED_PROPS.hasOwnProperty(propName)
    ) {
      if (config[propName] === undefined && defaultProps !== undefined) {
        props[propName] = defaultProps[propName];
      } else {
        props[propName] = config[propName];
      }
    }
  }
  // 处理新的 children
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }
  // 然后调用 ReactElement 方法来构建一个 Element 对象
  return ReactElement(element.type, key, ref, self, source, owner, props);
}
```

cloneElement 方法就是复制原有元素的信息,然后使用新的信息进行覆盖,最后返回一个新的 element 对象

### React.isValidElement

这个 api 是用来判断给定对象是否为 ReactElement 对象的非常简单实用的方法.

```js
export function isValidElement(object) {
  return (
    typeof object === "object" &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
}
```

### React.Children

React.Children 提供了处理 this.props.children 不透明数据结构的实用方法

#### React.Children.map

```ts
function mapChildren(
  children: ?ReactNodeList,
  func: MapFunc,
  context: mixed
): ?Array<React$Node> {
  if (children == null) {
    return children;
  }
  const result = [];
  let count = 0;
  mapIntoArray(children, result, "", "", function (child) {
    return func.call(context, child, count++);
  });
  return result;
}
```

可以看出 map 方法调用了 mapIntoArray 方法将 children 处理成数组放入结果数组中

#### mapIntoArray

mapIntoArray 方法接收 children 作为参数,并放入指定数组容器中

```ts
function mapIntoArray(
  children: ?ReactNodeList,
  array: Array<React$Node>,
  escapedPrefix: string,
  nameSoFar: string,
  callback: (?React$Node) => ?ReactNodeList,
): number{
  const type = typeof children;
  // 省略一些检查代码

  // 如果 children 是 字符串 数字 或者单个 element对象则 invokeCallback = true
  if (invokeCallback) {
    const child = children;
    let mappedChild = callback(child); // 调用回调取到返回值
     const childKey =
      nameSoFar === '' ? SEPARATOR + getElementKey(child, 0) : nameSoFar; // 生成一个 key
    if (Array.isArray(mappedChild)) { // 判断返回的是不是一个数组,如果是一个数组那么进行递归
      let escapedChildKey = '';
      if (childKey != null) {
        escapedChildKey = escapeUserProvidedKey(childKey) + '/';
      }
      mapIntoArray(mappedChild, array, escapedChildKey, '', c => c);
    } else if (mappedChild != null) {
      if (isValidElement(mappedChild)) {
        mappedChild = cloneAndReplaceKey( // cloneAndReplaceKey 是 ReactElement 的一个包装函数,具体制定了 element 对象的key
          mappedChild,
          // Keep both the (mapped) and old keys if they differ, just as
          // traverseAllChildren used to do for objects as children
          escapedPrefix +
            // $FlowFixMe Flow incorrectly thinks React.Portal doesn't have a key
            (mappedChild.key && (!child || child.key !== mappedChild.key)
              ? // $FlowFixMe Flow incorrectly thinks existing element's key can be a number
                escapeUserProvidedKey('' + mappedChild.key) + '/'
              : '') +
            childKey,
        );
      }
      array.push(mappedChild);
    }
    return 1;
  }

  // 如果 children 不是单个元素或者原始类型
  let child;
  let nextName;
  let subtreeCount = 0; // Count of children found in the current subtree.
  const nextNamePrefix =
    nameSoFar === '' ? SEPARATOR : nameSoFar + SUBSEPARATOR;

  if (Array.isArray(children)) { // 多个 children 会是一个数组
    for (let i = 0; i < children.length; i++) { // 然后遍历递归
      child = children[i];
      nextName = nextNamePrefix + getElementKey(child, i);
      subtreeCount += mapIntoArray(
        child,
        array,
        escapedPrefix,
        nextName,
        callback,
      );
    }
  } else { // 如果不是数组 可能是可迭代对象
    const iteratorFn = getIteratorFn(children);
    if (typeof iteratorFn === 'function') {
      const iterableChildren: Iterable<React$Node> & {
        entries: any,
      } = (children: any);
      const iterator = iteratorFn.call(iterableChildren);
      let step;
      let ii = 0;
      while (!(step = iterator.next()).done) {
        child = step.value;
        nextName = nextNamePrefix + getElementKey(child, ii++);
        subtreeCount += mapIntoArray(
          child,
          array,
          escapedPrefix,
          nextName,
          callback,
        );
      }
    }
  }// 省略一个 else
  return subtreeCount;
}

```

#### React.Children.forEach

函数很简单就是 map 函数的一个包装, 没有返回值

```ts
function forEachChildren(
  children: ?ReactNodeList,
  forEachFunc: ForEachFunc,
  forEachContext: mixed
): void {
  mapChildren(
    children,
    function () {
      forEachFunc.apply(this, arguments);
      // Don't return anything.
    },
    forEachContext
  );
}
```

#### React.Children.count

也是 map 的一个包装, 返回计数

```ts
function countChildren(children: ?ReactNodeList): number {
  let n = 0;
  mapChildren(children, () => {
    n++;
    // Don't return anything
  });
  return n;
}
```

#### React.Children.only

验证 children 是否只有一个子节点（一个 React 元素），如果有则返回它，否则此方法会抛出错误

```ts
function onlyChild<T>(children: T): T {
  invariant(
    isValidElement(children),
    "React.Children.only expected to receive a single React element child."
  );
  return children;
}
```

#### React.Children.toArray

也是 map 函数的一个包装

```ts
function toArray(children: ?ReactNodeList): Array<React$Node> {
  return mapChildren(children, (child) => child) || [];
}
```

## Fragments

[ReactSymbols](https://github.com/facebook/react/blob/master/packages/shared/ReactSymbols.js)

### React.Fragment

Fragment 比较简单就是一个 React 指定的标签.

```js
// like this
REACT_FRAGMENT_TYPE = symbolFor("react.fragment");
// or
export let REACT_FRAGMENT_TYPE = 0xeacb;
```

## Refs

在 React 中可以通过给组件添加 ref 属性来来获得:

- 如果组件是一个 class 组件那么 ref 将获得组件实例
- 如果组件是一个 dom 组件那么 ref 将获得实际 dom

### React.createRef

[ReactCreateRef](https://github.com/facebook/react/blob/master/packages/react/src/ReactCreateRef.js)

代码很简单就是返回了一个对象

```js
// an immutable object with a single mutable value
// 一个具有可变值的不可变对象
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  if (__DEV__) {
    Object.seal(refObject);
  }
  return refObject;
}
```

### React.forwardRef

React.forwardRef 会创建一个 React 组件, 这个组件能够将其接受的 ref 转发到其组件数的另一个组件中

```ts
export function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
  // 省略一些检查检查
  const elementType = {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };
  // 省略一些检查检查
  return elementType;
}
```

核心就是返回一个对象 \$\$typeof 值为 REACT_FORWARD_REF_TYPE 属性, 普通 React element 对象的值为 REACT_ELEMENT_TYPE

所以 React 通过给与这个属性不同的值来创建不同的独特元素,实现不同的功能.

## Suspense

### React.lazy

React.lazy() 允许你定义一个动态加载的组件。

```ts
export function lazy<T>(
  ctor: () => Thenable<{default: T, ...}>,
): LazyComponent<T, Payload<T>> {
  const payload: Payload<T> = {
    // We use these fields to store the result.
    // 我们使用这些组件来储存结果
    _status: -1,
    _result: ctor,
  };

  const lazyType: LazyComponent<T, Payload<T>> = {
    $$typeof: REACT_LAZY_TYPE, // 独特类型表明自己是 lazy 组件
    _payload: payload, // 信息存储对象
    _init: lazyInitializer, // lazyInitializer 是一个函数
  };

  // 省略一些开发检查
  return lazyType;
}
```

省略掉一些开发代码可以看到整体代码流程:

- 构造一个 payload 对象对象有两个属性 \_status 表明当前状态, \_result 就是 lazy 接收到的回调函数
- 构造一个 lazyType 对象, 同样的对象 \$\$typeof 属性表明了自己是一个 react lazy type
- lazyType 的 \_init 属性值是一个函数, 函数 lazyInitializer 被调用后会尝试执行回调函数得到函数的返回值进行执行,然后修改\_payload 对象的 \_status 属性和 \_result 属性,如果 Promise 函数执行错误会抛出错误

### React.Suspense

React.Suspense 是配合 lazy 使用的, suspense 也是一个 React 提供的类型

```js
// 支持 symbol
REACT_SUSPENSE_TYPE = symbolFor("react.suspense");
// or
REACT_SUSPENSE_TYPE = 0xead1;
```

## Hook

[ReactHooks](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js)
Hooks 是什么就不多做介绍了,文档上都有.

base hooks

在 React core 部分 Hooks 的是形式都如下

```ts
export function useHook() {
  const dispatcher = resolveDispatcher();
  return dispatcher.useHook();
}
```

都是通过 调用 resolveDispatcher 来获得一个 dispatcer 对象, 然后调用该对象上的方法

resolveDispatcher

```ts
function resolveDispatcher() {
  const dispatcher = ReactCurrentDispatcher.current;
  invariant(
    dispatcher !== null,
    "Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for" +
      " one of the following reasons:\n" +
      "1. You might have mismatching versions of React and the renderer (such as React DOM)\n" +
      "2. You might be breaking the Rules of Hooks\n" +
      "3. You might have more than one copy of React in the same app\n" +
      "See https://fb.me/react-invalid-hook-call for tips about how to debug and fix this problem."
  );
  return dispatcher;
}
```

ReactCurrentDispatcher 是一个不可变对象,current 由 实际渲染器对其赋值例如 ReactDOM
