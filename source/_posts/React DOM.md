---
title: React DOM
date: 2017-12-26 19:23:32
tags: React
---

React 学习

<!-- more -->

# ReactDOM

ReactDOM 是 React 的 DOM 和 server 的 `renderers`

## ReactDOM.render

```ts
export function render(
  element: React$Element<any>,
  container: Container,
  callback: ?Function
) {
  // 检查 container 是否为 DOM 元素
  invariant(
    isValidContainer(container),
    "Target container is not a DOM element."
  );
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback
  );
}
```

## legacyRenderSubtreeIntoContainer

通过函数可以看到 React 会在 Container DOM 上添加一个 \_reactRootContainer 属性, 值由 legacyCreateRootFromDOMContainer 函数的返回值

所以函数内会判断这个值是否存在如果存在的话就会使用他的值,如果不存在就调用函数创建它

从代码注释中可以得到, React 分为 mount 和 update 两个阶段

```ts
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: Container,
  forceHydrate: boolean,
  callback: ?Function,
) {
  // TODO: Without `any` type, Flow says "Property cannot be accessed on any
  // member of intersection type." Whyyyyyy.
  let root: RootType = (container._reactRootContainer: any);
  let fiberRoot;
  if (!root) {
    // Initial mount
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Initial mount should not be batched.
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Update
    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  return getPublicRootInstance(fiberRoot);
}
```

## Initial mount

### legacyCreateRootFromDOMContainer

函数本身功能比较单一, 出去检查是否为服务端渲染, 如果不是服务端渲染那么清除容器内部的所有元素.

```ts
function legacyCreateRootFromDOMContainer(
  container: Container,
  forceHydrate: boolean
): RootType {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // First clear any existing content.
  if (!shouldHydrate) {
    let warned = false;
    let rootSibling;
    while ((rootSibling = container.lastChild)) {
      // 省略检查代码
      container.removeChild(rootSibling);
    }
  }
  // 省略检查代码
  return createLegacyRoot(
    container,
    shouldHydrate
      ? {
          hydrate: true,
        }
      : undefined
  );
}
```

### createLegacyRoot

直接生成 ReactDOMBlockingRoot 实例然后返回

```ts
export function createLegacyRoot(
  container: Container,
  options?: RootOptions
): RootType {
  return new ReactDOMBlockingRoot(container, LegacyRoot, options);
}
```

```ts
function ReactDOMBlockingRoot(
  container: Container,
  tag: RootTag,
  options: void | RootOptions
) {
  this._internalRoot = createRootImpl(container, tag, options);
}
```

```ts
function createRootImpl(
  container: Container,
  tag: RootTag,
  options: void | RootOptions
) {
  // Tag is either LegacyRoot or Concurrent Root
  const hydrate = options != null && options.hydrate === true;
  const hydrationCallbacks =
    (options != null && options.hydrationOptions) || null;
  const root = createContainer(container, tag, hydrate, hydrationCallbacks);
  markContainerAsRoot(root.current, container); // 向 DOM 添加数据
  if (hydrate && tag !== LegacyRoot) {
    const doc =
      container.nodeType === DOCUMENT_NODE
        ? container
        : container.ownerDocument;
    eagerlyTrapReplayableEvents(container, doc);
  }
  return root;
}
```

createContainer 是 react-reconciler 包里面的方法

## react-reconciler

[document](https://github.com/facebook/react/blob/master/packages/react-reconciler/README.md)

### createContainer

createContainer 函数 return 了 createFiberRoot 函数

### createFiberRoot

```ts
export function createFiberRoot(
  containerInfo: any,
  tag: RootTag,
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,
): FiberRoot {
  // 会创建一个 FiberRoot
  const root: FiberRoot = (new FiberRootNode(containerInfo, tag, hydrate): any);
  if (enableSuspenseCallback) { // 开启新特性, 默认为false
    root.hydrationCallbacks = hydrationCallbacks;
  }

  // Cyclic construction. This cheats the type system right now because
  // stateNode is any.
  const uninitializedFiber = createHostRootFiber(tag); // tag 的值分别是 0 1 2 , 其中 0 是 16.13 默认模式 1 2 都是未来的试验模式
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

  initializeUpdateQueue(uninitializedFiber);

  return root;
}

```

这里出现了一个 FiberRootNode 这是 Fiber 里面的一个数据结构

### FiberRoot

```ts
type BaseFiberRootProperties = {|
  // The type of root (legacy, batched, concurrent, etc.)
  tag: RootTag, // 16.13 当前 RootTag React.render 使用的值为0 对应 legacy

  // Any additional information from the host associated with this root.
  // root 节点 DOM 容器
  containerInfo: any,
  // Used only by persistent updates.
  // 仅用于数据持久化 在 ReactDOM 中不会用到
  pendingChildren: any,
  // The currently active root fiber. This is the mutable root of the tree.
  // 当前活动的 root fiber, 是当前树的可变根
  current: Fiber,

  pingCache: WeakMap<Wakeable, Set<mixed>> | Map<Wakeable, Set<mixed>> | null,

  // A finished work-in-progress HostRoot that's ready to be committed.
  // 已经处理完成的 HostRoot 可以准备 commited
  finishedWork: Fiber | null,
  // Timeout handle returned by setTimeout. Used to cancel a pending timeout, if
  // it's superseded by a new one.
  //  setTimeout 返回的值
  timeoutHandle: TimeoutHandle | NoTimeout,
  // Top context object, used by renderSubtreeIntoContainer
  // 顶级 context 对象, 用于 renderSubtreeIntoContainer 函数
  context: Object | null,
  // 新的Context 如果有 用于替换 context
  pendingContext: Object | null,
  // Determines if we should attempt to hydrate on the initial mount
  // 服务端渲染使用
  +hydrate: boolean,
  // Node returned by Scheduler.scheduleCallback
  // Scheduler.scheduleCallback 函数的返回值
  callbackNode: *,

  // Only used by old reconciler
  // 仅用于旧的 reconciler
  // Expiration of the callback associated with this root
  // 与此 root 相关的回调截止时间
  callbackExpirationTime: ExpirationTime,
  // Priority of the callback associated with this root
  // 与此root相关的回调优先级
  callbackPriority: ReactPriorityLevel,// 普通为90 最高 99
  // 完成的截止时间
  finishedExpirationTime: ExpirationTime,
  // The earliest pending expiration time that exists in the tree
  // 当前树存在的最早过期时间
  firstPendingTime: ExpirationTime,
  // The latest pending expiration time that exists in the tree
  // 最晚过期时间
  lastPendingTime: ExpirationTime,
  // The earliest suspended expiration time that exists in the tree
  // 当前树存在的最早暂停时间
  firstSuspendedTime: ExpirationTime,
  // The latest suspended expiration time that exists in the tree
  // 最晚暂停时间
  lastSuspendedTime: ExpirationTime,
  // The next known expiration time after the suspended range
  // 下一个已知的 暂停时间范围的截止时间
  nextKnownPendingLevel: ExpirationTime,
  // The latest time at which a suspended component pinged the root to
  // render again
  lastPingedTime: ExpirationTime,
  lastExpiredTime: ExpirationTime,
  // Used by useMutableSource hook to avoid tearing within this root
  // when external, mutable sources are read from during render.
  mutableSourceLastPendingUpdateTime: ExpirationTime,

  // Only used by new reconciler
  // 仅用于 新的  reconciler
  // Represents the next task that the root should work on, or the current one
  // if it's already working.
  callbackId: Lanes,
  // Whether the currently scheduled task for this root is synchronous or
  // batched/concurrent. We have to track this because Scheduler does not
  // support synchronous tasks, so we put those on a separate queue. So you
  // could also think of this as "which queue is the callback scheduled with?"
  callbackIsSync: boolean,
  // Timestamp at which we will synchronously finish the current task to
  // prevent starvation.
  // TODO: There should be a separate expiration per lane.
  // NOTE: This is not an "ExpirationTime" as used by the old reconciler. It's a
  // timestamp, in milliseconds.
  expiresAt: number,

  pendingLanes: Lanes,
  suspendedLanes: Lanes,
  pingedLanes: Lanes,
  expiredLanes: Lanes,
  mutableReadLanes: Lanes,

  finishedLanes: Lanes,
|}
```

### Fiber Node

```ts
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
  // 识别 fiber 的类型 0 - 24
  // packages/react-reconciler/src/ReactWorkTags.js
  tag: WorkTag, // 函数组件是 0 class组件是 1 还没确定是函数还是类的时候是 2 HostRoot 是 3 其他参考文件

  // Unique identifier of this child.
  // key 唯一标示
  key: null | string,

  // The value of element.type which is used to preserve the identity during
  // reconciliation of this child.
  // react element 对象的 type 值.
  elementType: any,

  // The resolved function/class/ associated with this fiber.
  type: any,

  // The local state associated with this fiber.
  // 与 fiber 相关的本地实现
  stateNode: any,

  // Conceptual aliases
  // 概念别名
  // parent : Instance -> return The parent happens to be the same as the
  // return fiber since we've merged the fiber and instance.

  // Remaining fields belong to Fiber

  // The Fiber to return to after finishing processing this one.
  // This is effectively the parent, but there can be multiple parents (two)
  // so this is only the parent of the thing we're currently processing.
  // It is conceptually the same as the return address of a stack frame.
  // 父级的引用, 类似链表的 previous ,实际上就是父节点,但是可能有多个父节点, Fiber 加工完成之后通过此值返回
  return: Fiber | null,

  // Singly Linked List Tree Structure.
  // 单向链表的下一个Fiber对象
  child: Fiber | null,
  // 兄弟节点
  sibling: Fiber | null,
  index: number,

  // The ref last used to attach this node.
  // I'll avoid adding an owner field for prod and model that as functions.
  // ref
  ref:
    | null
    | (((handle: mixed) => void) & {_stringRef: ?string, ...})
    | RefObject,

  // Input is the data coming into process this fiber. Arguments. Props.
  pendingProps: any, // This type will be more specific once we overload the tag.
  memoizedProps: any, // The props used to create the output.

  // A queue of state updates and callbacks.
  updateQueue: mixed,

  // The state used to create the output
  memoizedState: any,

  // Dependencies (contexts, events) for this fiber, if it has any
  dependencies_new: Dependencies_new | null,
  dependencies_old: Dependencies_old | null,

  // Bitfield that describes properties about the fiber and its subtree. E.g.
  // the ConcurrentMode flag indicates whether the subtree should be async-by-
  // default. When a fiber is created, it inherits the mode of its
  // parent. Additional flags can be set at creation time, but after that the
  // value should remain unchanged throughout the fiber's lifetime, particularly
  // before its child fibers are created.
  mode: TypeOfMode, // packages/react-reconciler/src/ReactTypeOfMode.js 几种模式

  // Effect
  effectTag: SideEffectTag,

  // Singly linked list fast path to the next fiber with side-effects.
  nextEffect: Fiber | null,

  // The first and last fiber with side-effect within this subtree. This allows
  // us to reuse a slice of the linked list when we reuse the work done within
  // this fiber.
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,

  // Only used by old reconciler
  expirationTime: ExpirationTime,
  childExpirationTime: ExpirationTime,

  // Only used by new reconciler
  lanes: Lanes,
  childLanes: Lanes,

  // This is a pooled version of a Fiber. Every fiber that gets updated will
  // eventually have a pair. There are cases when we can clean up pairs to save
  // memory if we need to.
  alternate: Fiber | null,

  // Time spent rendering this Fiber and its descendants for the current update.
  // This tells us how well the tree makes use of sCU for memoization.
  // It is reset to 0 each time we render and only updated when we don't bailout.
  // This field is only set when the enableProfilerTimer flag is enabled.
  // 渲染这棵树和他的后代所花费的时间.
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
  _debugHookTypes?: Array<HookType> | null,
|};

type BaseFiberRootProperties = {|
  // The type of root (legacy, batched, concurrent, etc.)
  tag: RootTag,

  // Any additional information from the host associated with this root.
  containerInfo: any,
  // Used only by persistent updates.
  pendingChildren: any,
  // The currently active root fiber. This is the mutable root of the tree.
  current: Fiber,

  pingCache: WeakMap<Wakeable, Set<mixed>> | Map<Wakeable, Set<mixed>> | null,

  // A finished work-in-progress HostRoot that's ready to be committed.
  finishedWork: Fiber | null,
  // Timeout handle returned by setTimeout. Used to cancel a pending timeout, if
  // it's superseded by a new one.
  timeoutHandle: TimeoutHandle | NoTimeout,
  // Top context object, used by renderSubtreeIntoContainer
  context: Object | null,
  pendingContext: Object | null,
  // Determines if we should attempt to hydrate on the initial mount
  +hydrate: boolean,
  // Node returned by Scheduler.scheduleCallback
  callbackNode: *,

  // Only used by old reconciler

  // Expiration of the callback associated with this root
  callbackExpirationTime: ExpirationTime,
  // Priority of the callback associated with this root
  callbackPriority: ReactPriorityLevel,

  finishedExpirationTime: ExpirationTime,
  // The earliest pending expiration time that exists in the tree
  firstPendingTime: ExpirationTime,
  // The latest pending expiration time that exists in the tree
  lastPendingTime: ExpirationTime,
  // The earliest suspended expiration time that exists in the tree
  firstSuspendedTime: ExpirationTime,
  // The latest suspended expiration time that exists in the tree
  lastSuspendedTime: ExpirationTime,
  // The next known expiration time after the suspended range
  nextKnownPendingLevel: ExpirationTime,
  // The latest time at which a suspended component pinged the root to
  // render again
  lastPingedTime: ExpirationTime,
  lastExpiredTime: ExpirationTime,
  // Used by useMutableSource hook to avoid tearing within this root
  // when external, mutable sources are read from during render.
  mutableSourceLastPendingUpdateTime: ExpirationTime,

  // Only used by new reconciler

  // Represents the next task that the root should work on, or the current one
  // if it's already working.
  callbackId: Lanes,
  // Whether the currently scheduled task for this root is synchronous or
  // batched/concurrent. We have to track this because Scheduler does not
  // support synchronous tasks, so we put those on a separate queue. So you
  // could also think of this as "which queue is the callback scheduled with?"
  callbackIsSync: boolean,
  // Timestamp at which we will synchronously finish the current task to
  // prevent starvation.
  // TODO: There should be a separate expiration per lane.
  // NOTE: This is not an "ExpirationTime" as used by the old reconciler. It's a
  // timestamp, in milliseconds.
  expiresAt: number,

  pendingLanes: Lanes,
  suspendedLanes: Lanes,
  pingedLanes: Lanes,
  expiredLanes: Lanes,
  mutableReadLanes: Lanes,

  finishedLanes: Lanes,
|};

```

自此 初始化阶段完毕, FiberRoot 和 RootFiber 都创建完毕接下来进入初次更新阶段

## Initial Update

### unbatchedUpdates

legacyRenderSubtreeIntoContainer 函数中的

```ts
// Initial mount should not be batched.
unbatchedUpdates(() => {
  // childen 为 React.render 第一个参数 React element 对象
  // fiberRoot 是在上面创建了.仅仅是一个 FiberRoot 和它的 RootFiber
  updateContainer(children, fiberRoot, parentComponent, callback);
});
```

executionContext 表明了当前正处于什么上下文,目前有七个:

```ts
const NoContext = /*                    */ 0b000000; 
const BatchedContext = /*               */ 0b000001; 
const EventContext = /*                 */ 0b000010; 
const DiscreteEventContext = /*         */ 0b000100; 
const LegacyUnbatchedContext = /*       */ 0b001000; 
const RenderContext = /*                */ 0b010000; 
const CommitContext = /*                */ 0b100000;

```

```ts
export function unbatchedUpdates<A, R>(fn: (a: A) => R, a: A): R {
  const prevExecutionContext = executionContext;
  executionContext &= BatchedContext~;
  executionContext |= LegacyUnbatchedContext;
  try {
    return fn(a);
  } finally {
    executionContext = prevExecutionContext;
    if (executionContext === NoContext) {
      // Flush the immediate callbacks that were scheduled during this batch
      flushSyncCallbackQueue();
    }
  }
}
```

### unbatchedUpdates
