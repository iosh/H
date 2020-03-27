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

## ReactDOM.render

函数签名

```jsx
function render(
  element: React$Element<any>,
  container: Container,
  callback: ?Function
) {}
```

函数做了一些检查,例如检查 container 是否为 DOM 容器, 之后会调用 legacyRenderSubtreeIntoContainer

### legacyRenderSubtreeIntoContainer

```jsx
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>, // 指定父容器, 用于 ReactDOM.unstable_renderSubtreeIntoContainer 这个api
  children: ReactNodeList, // node tree
  container: Container, // 容器
  forceHydrate: boolean, // 服务端渲染
  callback: ?Function
) {}
```

之后函数内部开始生成 root 节点, 这个时候的 children 还是 createElement 生成的描述对象

- Initial mount

函数开始会检查当前 container (DOM 元素上是否有指定标记,以确定是否为第一次渲染).

没有则会调用 legacyCreateRootFromDOMContainer 函数生成这个对象.

legacyCreateRootFromDOMContainer

函数首先会清除给定 DOM 的所有已存在的元素

然后调用 createLegacyRoot ,而这个函数直接 new ReactDOMBlockingRoot(container, LegacyRoot, options)

ReactDOMBlockingRoot 是一个构造函数有三个主要的东西:

1. \_\_internalRoot
   这个属性的值是由 createContainer 函数调用 createFiberRoot 函数创建的, 生成一个 FiberRootNode 并为其 current 生成一个 Fiber 对象.并且初始化了这个 Fiber 的 updateQueue

2. render 函数
   该函数直接调用了 updateContainer 函数
3. unmount 函数
   该函数也调用了 updateContainer 函数并传递而一个回调函数,用于去掉 root DOM 上储存的信息

### FiberRoot

React 源代码中使用了 flow 来标注类型,

```tsx
type BaseFiberRootProperties = {|
  tag: RootTag,// The type of root (legacy, batched, concurrent, etc.)
  containerInfo: any,// Any additional information from the host associated with this root.
  pendingChildren: any, // used only by persistent update  仅在用于持久更新
  // The currently active root fiber. This is the mutable root of the tree.
  current: Fiber,

  // 通过搜索得知是一个 WeakMap 具体作用不明
  pingCache:
    | WeakMap<Thenable, Set<ExpirationTime>>
    | Map<Thenable, Set<ExpirationTime>>
    | null,

  // 一个超时时间
  finishedExpirationTime: ExpirationTime,
  // 一个 fiber 对象,在commit阶段会被处理
  finishedWork: Fiber | null,
 // 任务被挂起的时候 setTimeout 的返回值,在新的任务执行的时候用它清理未执行的定时器
  timeoutHandle: TimeoutHandle | NoTimeout,
  // 顶级 context 对象, 被 renderSubtereeIntoContainer 函数使用
  context: Object | null,
  pendingContext: Object | null,
  // 是否为服务端渲染,用来确定是否需要融合
  +hydrate: boolean,
  // Scheduler.scheduleCallback 函数返回
  callbackNode: *,
  // 当前 root 过期时间的回调
  callbackExpirationTime: ExpirationTime,
  // 当前 root 优先级回调
  callbackPriority: ReactPriorityLevel,
  // 当前树存在的最早挂起时间
  firstPendingTime: ExpirationTime,
  //
  firstSuspendedTime: ExpirationTime,
  //
  lastSuspendedTime: ExpirationTime,
  //
  nextKnownPendingLevel: ExpirationTime,
  //
  //
  lastPingedTime: ExpirationTime,
  lastExpiredTime: ExpirationTime,
  //
  //
  mutableSourcePendingUpdateTime: ExpirationTime,
|}
 // React DevTools 插件相关
type ProfilingOnlyFiberRootProperties = {|
  interactionThreadID: number,
  memoizedInteractions: Set<Interaction>,
  pendingInteractionMap: PendingInteractionMap,
|};
// 服务端渲染相关
type SuspenseCallbackOnlyFiberRootProperties = {|
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
|};
```

### Fiber

```jsx
// A Fiber is work on a Component that needs to be done or was done. There can
// be more than one per component.
// Fiber 是每个组件需要做的或者做完的工作, 一个组件可能含有不止一个 Fiber 对象
export type Fiber = {|
  // These first fields are conceptually members of an Instance. This used to
  // be split into a separate type and intersected with the other Fiber fields,
  // but until Flow fixes its intersection bugs, we've merged them into a
  // single type.

  // An Instance is shared between all versions of a component. We can easily
  // break this out into a separate object to avoid copying so much to the
  // alternate versions of the tree. We put this on a single object for now to
  // minimize the number of objects created during the initial render.

  // Tag identifying the type of fiber.
  tag: WorkTag, //组件类型标记

  // Unique identifier of this child.
  key: null | string, // key

  // The value of element.type which is used to preserve the identity during
  // reconciliation of this child.
  elementType: any, // createElement 的第一个参数,

  // The resolved function/class/ associated with this fiber.
  type: any,

  // The local state associated with this fiber.
  // FiberRootNode 对象,具体定义看上面.
  stateNode: any,

  // Conceptual aliases
  // parent : Instance -> return The parent happens to be the same as the
  // return fiber since we've merged the fiber and instance.

  // Remaining fields belong to Fiber

  // The Fiber to return to after finishing processing this one.
  // This is effectively the parent, but there can be multiple parents (two)
  // so this is only the parent of the thing we're currently processing.
  // It is conceptually the same as the return address of a stack frame.
  // 父节点的 Fiber 对象
  return: Fiber | null,

  // Singly Linked List Tree Structure.
  child: Fiber | null,
  sibling: Fiber | null,
  index: number,

  // The ref last used to attach this node.
  // I'll avoid adding an owner field for prod and model that as functions.
  ref:
    | null
    | (((handle: mixed) => void) & { _stringRef: ?string, ... })
    | RefObject,

  // Input is the data coming into process this fiber. Arguments. Props.
  pendingProps: any, // This type will be more specific once we overload the tag.
  memoizedProps: any, // The props used to create the output.

  // A queue of state updates and callbacks.
  // 状态更新和回调队列
  updateQueue: UpdateQueue<any> | null,

  // The state used to create the output
  memoizedState: any,

  // Dependencies (contexts, events) for this fiber, if it has any
  // 当前 fiber 的一些依赖如果有
  dependencies: Dependencies | null,

  // Bitfield that describes properties about the fiber and its subtree. E.g.
  // the ConcurrentMode flag indicates whether the subtree should be async-by-
  // default. When a fiber is created, it inherits the mode of its
  // parent. Additional flags can be set at creation time, but after that the
  // value should remain unchanged throughout the fiber's lifetime, particularly
  // before its child fibers are created.
  // 用来标书当前 Fiber 和他的 subtree 的 Bitfield, 通常继承父级, 创建成功后不会修改(主要是描述渲染方式,当前的普通还是未来的异步)
  mode: TypeOfMode,

  // Effect
  effectTag: SideEffectTag,

  // Singly linked list fast path to the next fiber with side-effects.
  nextEffect: Fiber | null,

  // The first and last fiber with side-effect within this subtree. This allows
  // us to reuse a slice of the linked list when we reuse the work done within
  // this fiber.
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,

  // Represents a time in the future by which this work should be completed.
  // Does not include work found in its subtree.
  expirationTime: ExpirationTime,

  // This is used to quickly determine if a subtree has no pending changes.
  childExpirationTime: ExpirationTime,

  // This is a pooled version of a Fiber. Every fiber that gets updated will
  // eventually have a pair. There are cases when we can clean up pairs to save
  // memory if we need to.
  // 用来合并的 Fiber, 每个更新 fiber 都会有
  alternate: Fiber | null,

  // Time spent rendering this Fiber and its descendants for the current update.
  // This tells us how well the tree makes use of sCU for memoization.
  // It is reset to 0 each time we render and only updated when we don't bailout.
  // This field is only set when the enableProfilerTimer flag is enabled.
  actualDuration?: number,

  // If the Fiber is currently active in the "render" phase,
  // This marks the time at which the work began.
  // This field is only set when the enableProfilerTimer flag is enabled.
  actualStartTime?: number,

  // Duration of the most recent render time for this Fiber.
  // This value is not updated when we bailout for memoization purposes.
  // This field is only set when the enableProfilerTimer flag is enabled.
  selfBaseDuration?: number,

  // Sum of base times for all descendants of this Fiber.
  // This value bubbles up during the "complete" phase.
  // This field is only set when the enableProfilerTimer flag is enabled.
  treeBaseDuration?: number,

  // Conceptual aliases
  // workInProgress : Fiber ->  alternate The alternate used for reuse happens
  // to be the same as work in progress.
  // __DEV__ only
  _debugID?: number,
  _debugSource?: Source | null,
  _debugOwner?: Fiber | null,
  _debugIsCurrentlyTiming?: boolean,
  _debugNeedsRemount?: boolean,

  // Used to verify that the order of hooks does not change between renders.
  _debugHookTypes?: Array<HookType> | null
|};
```

### Fiber.updateQueue

```tsx
type UpdateQueue<State> = {|
  baseState: State,
  firstBaseUpdate: Update<State> | null,
  lastBaseUpdate: Update<State> | null,
  shared: SharedQueue<State>,
  effects: Array<Update<State>> | null,
|}
```

### Update

```tsx
export type Update<State> = {|
  expirationTime: ExpirationTime,
  suspenseConfig: null | SuspenseConfig,

  tag: 0 | 1 | 2 | 3,
  payload: any,
  callback: (() => mixed) | null,

  next: Update<State> | null,

  // DEV only
  priority?: ReactPriorityLevel,
|};

```

第一次渲染是直接使用 unbatchedUpdates 函数然后将 updateContainer 函数所谓回调函数传入, 然后在 try catch 内执行 updateContainer 函数, 最后 调用 flushSyncCallbackQueue 函数

### unbatchedUpdates

函数记录当前执行 executionContext, 从 NoContext 变化到 LegacyUnbatchedContext, 执行完 updateContainer 之后再讲状变回去.

### updateContainer

函数 updateContainer 调用 热区 setCurrentTimeForUpdate

而 requestCurrentTimeForUpdate 调用了 msToExpirationTime 来从时间戳获得 currentTime

```jsx
//  ms 参数 来源有两个 如果当前浏览器支持 performance.now() 就使用,如果不支持就使用 Date().now()
// performance.now  to see https://developers.google.com/web/updates/2012/08/When-milliseconds-are-not-enough-performance-now?hl=en

// 1 unit of expiration time represents 10ms.
// / UNIT_SIZE (值为10) 是为了抹平 10ms内的误差, 10ms内都是为一致
export function msToExpirationTime(ms: number): ExpirationTime {
  // Always subtract from the offset so that we don't clash with the magic number for NoWork.
  // MAGIC_NUMBER_OFFSET 的值是 32位二进制最大值 - 2
  // 最后的 | 0 是将 ms 强制转换为 32位,因为 Date.now() 获得的值必然超过 32位能表示的最大值,所以用 | 0 转换
  return MAGIC_NUMBER_OFFSET - ((ms / UNIT_SIZE) | 0);
}

// 这个函数是上面的逆运算将过期时间转换为 ms
export function expirationTimeToMs(expirationTime: ExpirationTime): number {
  return (MAGIC_NUMBER_OFFSET - expirationTime) * UNIT_SIZE;
}
```

之后调用 computeExpirationForFiber 计算 expirationTIme

computeExpirationForFiber 会判断当前是否为 同步模式(还有新的异步模式),如果是同步模式直接返回了最大 31 bit int

然后初始化一个 空的 对象作为 context 赋值给 FiberRootNode

然后调用 createUpdate 函数生成一个 update 对象

```jsx
export type Update<State> = {|
  expirationTime: ExpirationTime,
  suspenseConfig: null | SuspenseConfig,

  tag: 0 | 1 | 2 | 3,
  payload: any,
  callback: (() => mixed) | null,

  next: Update<State> | null,

  // DEV only
  priority?: ReactPriorityLevel
|};
```

之后就开始调用 enqueueUpdate 该函数就是把 fiber 对象上的 updateQueue 对象处理和赋值了一下.

之后调用 scheduleUpdateOnFiber 函数内检查了更新时间,并且将值为 0(初次更新)的值更新为上面获取的时间

之后调用了 createWorkInprogress 函数创建了当前 Fiber 的一个副本, 互相关联引用

```jsx
workInProgress.alternate = current;
current.alternate = workInProgress;
```

源码中带注释的一些信息


```jsx

// Describes where we are in the React execution stack
let executionContext: ExecutionContext = NoContext;
// The root we're working on
let workInProgressRoot: FiberRoot | null = null;
// The fiber we're working on
let workInProgress: Fiber | null = null;
// The expiration time we're rendering
let renderExpirationTime: ExpirationTime = NoWork;
// Whether to root completed, errored, suspended, etc.
let workInProgressRootExitStatus: RootExitStatus = RootIncomplete;
// A fatal error, if one is thrown
let workInProgressRootFatalError: mixed = null;
// Most recent event time among processed updates during this render.
// This is conceptually a time stamp but expressed in terms of an ExpirationTime
// because we deal mostly with expiration times in the hot path, so this avoids
// the conversion happening in the hot path.
let workInProgressRootLatestProcessedExpirationTime: ExpirationTime = Sync;
let workInProgressRootLatestSuspenseTimeout: ExpirationTime = Sync;
let workInProgressRootCanSuspendUsingConfig: null | SuspenseConfig = null;
// The work left over by components that were visited during this render. Only
// includes unprocessed updates, not work in bailed out children.
let workInProgressRootNextUnprocessedUpdateTime: ExpirationTime = NoWork;

// If we're pinged while rendering we don't always restart immediately.
// This flag determines if it might be worthwhile to restart if an opportunity
// happens latere.
let workInProgressRootHasPendingPing: boolean = false;
// The most recent time we committed a fallback. This lets us ensure a train
// model where we don't commit new loading states in too quick succession.

```


