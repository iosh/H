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

unbatchedUpdates 函数设定当前状态,然后运行了 updateContainer 函数, 并且使用了 try finally 函数在执行完成之后还原了原来的状态

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

### updateContainer

```ts
export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function
): ExpirationTime {
  const current = container.current; // get fiber root node
  const currentTime = requestCurrentTimeForUpdate(); // 内部进行一些判断, 然后调用msToExpirationTime 返回一个时间戳

  const suspenseConfig = requestCurrentSuspenseConfig();
  const expirationTime = computeExpirationForFiber(
    currentTime,
    current,
    suspenseConfig
  );

  const context = getContextForSubtree(parentComponent);
  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }
  // 省略一些检查代码
  const update = createUpdate(expirationTime, suspenseConfig);
  // Caution: React DevTools currently depends on this property
  // being called "element".
  update.payload = { element };

  callback = callback === undefined ? null : callback;
  // 省略一下检查代码
  enqueueUpdate(current, update); // 为当前 Fiber 对象构建一个 updateQueue 对象
  // 更新了一些Fiber上的过期时间
  // 然后调用 performSyncWorkOnRoot 同步更新
  scheduleUpdateOnFiber(current, expirationTime);

  return expirationTime;
}
```

### performSyncWorkOnRoot

```ts
// This is the entry point for synchronous tasks that don't go
// through Scheduler
// 同步任务切入点
function performSyncWorkOnRoot(root) {
  invariant(
    (executionContext & (RenderContext | CommitContext)) === NoContext,
    'Should not already be working.',
  );

  flushPassiveEffects();

  const lastExpiredTime = root.lastExpiredTime;

  let expirationTime;
  if (lastExpiredTime !== NoWork) {
    // There's expired work on this root. Check if we have a partial tree
    // that we can reuse.
    if (
      root === workInProgressRoot &&
      renderExpirationTime >= lastExpiredTime
    ) {
      // There's a partial tree with equal or greater than priority than the
      // expired level. Finish rendering it before rendering the rest of the
      // expired work.
      expirationTime = renderExpirationTime;
    } else {
      // Start a fresh tree.
      expirationTime = lastExpiredTime;
    }
  } else {
    // There's no expired work. This must be a new, synchronous render.
    expirationTime = Sync;
  }

  let exitStatus = renderRootSync(root, expirationTime);

  if (root.tag !== LegacyRoot && exitStatus === RootErrored) {
    // If something threw an error, try rendering one more time. We'll
    // render synchronously to block concurrent data mutations, and we'll
    // render at Idle (or lower) so that all pending updates are included.
    // If it still fails after the second attempt, we'll give up and commit
    // the resulting tree.
    expirationTime = expirationTime > Idle ? Idle : expirationTime;
    //
    exitStatus = renderRootSync(root, expirationTime);
  }

  if (exitStatus === RootFatalErrored) {
    const fatalError = workInProgressRootFatalError;
    prepareFreshStack(root, expirationTime);
    markRootSuspendedAtTime(root, expirationTime);
    ensureRootIsScheduled(root);
    throw fatalError;
  }

  // We now have a consistent tree. Because this is a sync render, we
  // will commit it even if something suspended.
  const finishedWork: Fiber = (root.current.alternate: any);
  root.finishedWork = finishedWork;
  root.finishedExpirationTime = expirationTime;
  root.nextKnownPendingLevel = getRemainingExpirationTime(finishedWork);
  commitRoot(root);

  // Before exiting, make sure there's a callback scheduled for the next
  // pending level.
  ensureRootIsScheduled(root);

  return null;
}
```

### renderRootSync

```ts
function renderRootSync(root, expirationTime) {
  const prevExecutionContext = executionContext;
  executionContext |= RenderContext; // 调整当前上下文为 renderContext
  const prevDispatcher = pushDispatcher(root); // 获得并且给 React 部分赋值 Hooks 钩子

  // If the root or expiration time have changed, throw out the existing stack
  // and prepare a fresh one. Otherwise we'll continue where we left off.
  if (root !== workInProgressRoot || expirationTime !== renderExpirationTime) {
    // prepareFreshStack 函数会判断 FiberRoot 是否有  alternate 属性 如果没有那么 调用  createWorkInProgress clone
    // root.alternate === workInProgress , workInProgress.alternate === root
    prepareFreshStack(root, expirationTime);
    startWorkOnPendingInteractions(root, expirationTime);
  }
  const prevInteractions = pushInteractions(root);

  do {
    try {
      workLoopSync(); // work work work :)
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  resetContextDependencies();
  if (enableSchedulerTracing) {
    popInteractions(((prevInteractions: any): Set<Interaction>));
  }

  executionContext = prevExecutionContext;
  popDispatcher(prevDispatcher);

  if (workInProgress !== null) {
    // This is a sync render, so we should have finished the whole tree.
    invariant(
      false,
      'Cannot commit an incomplete root. This error is likely caused by a ' +
        'bug in React. Please file an issue.',
    );
  }

  if (__DEV__) {
    if (enableDebugTracing) {
      logRenderStopped();
    }
  }

  // Set this to null to indicate there's no in-progress render.
  workInProgressRoot = null;

  return workInProgressRootExitStatus;
}
```

### workLoopSync

workLoopSync 会调用 performUnitOfWork 函数

### performUnitOfWork 函数

```ts
function performUnitOfWork(unitOfWork: Fiber): void {
  // The current, flushed, state of this fiber is the alternate. Ideally
  // nothing should rely on this, but relying on it here means that we don't
  // need an additional field on the work in progress.
  const current = unitOfWork.alternate; // 通过 alternate 取到  FiberRoot对象
  setCurrentDebugFiberInDEV(unitOfWork);

  let next;
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    next = beginWork(current, unitOfWork, renderExpirationTime);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, renderExpirationTime);
  }

  resetCurrentDebugFiberInDEV();
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  if (next === null) {
    // If this doesn't spawn new work, complete the current work.
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }

  ReactCurrentOwner.current = null;
}
```

### beginWork

```ts
let beginWork;
if (__DEV__ && replayFailedUnitOfWorkWithInvokeGuardedCallback) {
  const dummyFiber = null;
  beginWork = (current, unitOfWork, expirationTime) => {
    // If a component throws an error, we replay it again in a synchronously
    // dispatched event, so that the debugger will treat it as an uncaught
    // error See ReactErrorUtils for more information.
    // 如果以组件抛出一个错误, 我们可以在调度事件中重新进行一次, 浏览器会将其视为错误

    // Before entering the begin phase, copy the work-in-progress onto a dummy
    // fiber. If beginWork throws, we'll use this to reset the state.
    // 开始之前先将对象复制一下预防出错,出错后可以还原, 开发下行为
    const originalWorkInProgressCopy = assignFiberPropertiesInDEV(
      dummyFiber,
      unitOfWork,
    );
    try {
      return originalBeginWork(current, unitOfWork, expirationTime);
    } catch (originalError) {
      if (
        originalError !== null &&
        typeof originalError === 'object' &&
        typeof originalError.then === 'function'
      ) {
        // Don't replay promises. Treat everything else like an error.
        throw originalError;
      }

      // Keep this code in sync with handleError; any changes here must have
      // corresponding changes there.
      resetContextDependencies();
      resetHooksAfterThrow();
      // Don't reset current debug fiber, since we're about to work on the
      // same fiber again.

      // Unwind the failed stack frame
      unwindInterruptedWork(unitOfWork);

      // Restore the original properties of the fiber.
      assignFiberPropertiesInDEV(unitOfWork, originalWorkInProgressCopy);

      if (enableProfilerTimer && unitOfWork.mode & ProfileMode) {
        // Reset the profiler timer.
        startProfilerTimer(unitOfWork);
      }

      // Run beginWork again.
      invokeGuardedCallback(
        null,
        originalBeginWork,
        null,
        current,
        unitOfWork,
        expirationTime,
      );

      if (hasCaughtError()) {
        const replayError = clearCaughtError();
        // `invokeGuardedCallback` sometimes sets an expando `_suppressLogging`.
        // Rethrow this error instead of the original one.
        throw replayError;
      } else {
        // This branch is reachable if the render phase is impure.
        throw originalError;
      }
    }
  };
} else


```

### originalBeginWork

这个流程就是将 Fiber 树构建出来, 并且将元素也初始化出来下一步就是直接 commit 到 DOM 了

```ts
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderExpirationTime: ExpirationTime,
): Fiber | null {
  const updateExpirationTime = workInProgress.expirationTime;

  if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;

    if (
      oldProps !== newProps ||
      hasLegacyContextChanged() ||
      // Force a re-render if the implementation changed due to hot reload:
      (__DEV__ ? workInProgress.type !== current.type : false)
    ) {
      // If props or context changed, mark the fiber as having performed work.
      // This may be unset if the props are determined to be equal later (memo).
      didReceiveUpdate = true;
    } else if (updateExpirationTime < renderExpirationTime) {
      didReceiveUpdate = false;
      // This fiber does not have any pending work. Bailout without entering
      // the begin phase. There's still some bookkeeping we that needs to be done
      // in this optimized path, mostly pushing stuff onto the stack.
      switch (workInProgress.tag) {
        case HostRoot:
          pushHostRootContext(workInProgress);
          resetHydrationState();
          break;
        case HostComponent:
          pushHostContext(workInProgress);
          if (
            workInProgress.mode & ConcurrentMode &&
            renderExpirationTime !== Never &&
            shouldDeprioritizeSubtree(workInProgress.type, newProps)
          ) {
            if (enableSchedulerTracing) {
              markSpawnedWork(Never);
            }
            // Schedule this fiber to re-render at offscreen priority. Then bailout.
            workInProgress.expirationTime = workInProgress.childExpirationTime = Never;
            return null;
          }
          break;
        case ClassComponent: {
          const Component = workInProgress.type;
          if (isLegacyContextProvider(Component)) {
            pushLegacyContextProvider(workInProgress);
          }
          break;
        }
        case HostPortal:
          pushHostContainer(
            workInProgress,
            workInProgress.stateNode.containerInfo,
          );
          break;
        case ContextProvider: {
          const newValue = workInProgress.memoizedProps.value;
          pushProvider(workInProgress, newValue);
          break;
        }
        case Profiler:
          if (enableProfilerTimer) {
            // Profiler should only call onRender when one of its descendants actually rendered.
            const hasChildWork =
              workInProgress.childExpirationTime >= renderExpirationTime;
            if (hasChildWork) {
              workInProgress.effectTag |= Update;
            }

            // Reset effect durations for the next eventual effect phase.
            // These are reset during render to allow the DevTools commit hook a chance to read them,
            const stateNode = workInProgress.stateNode;
            stateNode.effectDuration = 0;
            stateNode.passiveEffectDuration = 0;
          }
          break;
        case SuspenseComponent: {
          const state: SuspenseState | null = workInProgress.memoizedState;
          if (state !== null) {
            if (enableSuspenseServerRenderer) {
              if (state.dehydrated !== null) {
                pushSuspenseContext(
                  workInProgress,
                  setDefaultShallowSuspenseContext(suspenseStackCursor.current),
                );
                // We know that this component will suspend again because if it has
                // been unsuspended it has committed as a resolved Suspense component.
                // If it needs to be retried, it should have work scheduled on it.
                workInProgress.effectTag |= DidCapture;
                break;
              }
            }

            // If this boundary is currently timed out, we need to decide
            // whether to retry the primary children, or to skip over it and
            // go straight to the fallback. Check the priority of the primary
            // child fragment.
            const primaryChildFragment: Fiber = (workInProgress.child: any);
            const primaryChildExpirationTime =
              primaryChildFragment.childExpirationTime;
            if (primaryChildExpirationTime >= renderExpirationTime) {
              // The primary children have pending work. Use the normal path
              // to attempt to render the primary children again.
              return updateSuspenseComponent(
                current,
                workInProgress,
                renderExpirationTime,
              );
            } else {
              // The primary child fragment does not have pending work marked
              // on it...

              // ...usually. There's an unfortunate edge case where the fragment
              // fiber is not part of the return path of the children, so when
              // an update happens, the fragment doesn't get marked during
              // setState. This is something we should consider addressing when
              // we refactor the Fiber data structure. (There's a test with more
              // details; to find it, comment out the following block and see
              // which one fails.)
              //
              // As a workaround, we need to recompute the `childExpirationTime`
              // by bubbling it up from the next level of children. This is
              // based on similar logic in `resetChildExpirationTime`.
              let primaryChild = primaryChildFragment.child;
              while (primaryChild !== null) {
                const childUpdateExpirationTime = primaryChild.expirationTime;
                const childChildExpirationTime =
                  primaryChild.childExpirationTime;
                if (
                  childUpdateExpirationTime >= renderExpirationTime ||
                  childChildExpirationTime >= renderExpirationTime
                ) {
                  // Found a child with an update with sufficient priority.
                  // Use the normal path to render the primary children again.
                  return updateSuspenseComponent(
                    current,
                    workInProgress,
                    renderExpirationTime,
                  );
                }
                primaryChild = primaryChild.sibling;
              }

              pushSuspenseContext(
                workInProgress,
                setDefaultShallowSuspenseContext(suspenseStackCursor.current),
              );
              // The primary children do not have pending work with sufficient
              // priority. Bailout.
              const child = bailoutOnAlreadyFinishedWork(
                current,
                workInProgress,
                renderExpirationTime,
              );
              if (child !== null) {
                // The fallback children have pending work. Skip over the
                // primary children and work on the fallback.
                return child.sibling;
              } else {
                return null;
              }
            }
          } else {
            pushSuspenseContext(
              workInProgress,
              setDefaultShallowSuspenseContext(suspenseStackCursor.current),
            );
          }
          break;
        }
        case SuspenseListComponent: {
          const didSuspendBefore =
            (current.effectTag & DidCapture) !== NoEffect;

          const hasChildWork =
            workInProgress.childExpirationTime >= renderExpirationTime;

          if (didSuspendBefore) {
            if (hasChildWork) {
              // If something was in fallback state last time, and we have all the
              // same children then we're still in progressive loading state.
              // Something might get unblocked by state updates or retries in the
              // tree which will affect the tail. So we need to use the normal
              // path to compute the correct tail.
              return updateSuspenseListComponent(
                current,
                workInProgress,
                renderExpirationTime,
              );
            }
            // If none of the children had any work, that means that none of
            // them got retried so they'll still be blocked in the same way
            // as before. We can fast bail out.
            workInProgress.effectTag |= DidCapture;
          }

          // If nothing suspended before and we're rendering the same children,
          // then the tail doesn't matter. Anything new that suspends will work
          // in the "together" mode, so we can continue from the state we had.
          const renderState = workInProgress.memoizedState;
          if (renderState !== null) {
            // Reset to the "together" mode in case we've started a different
            // update in the past but didn't complete it.
            renderState.rendering = null;
            renderState.tail = null;
            renderState.lastEffect = null;
          }
          pushSuspenseContext(workInProgress, suspenseStackCursor.current);

          if (hasChildWork) {
            break;
          } else {
            // If none of the children had any work, that means that none of
            // them got retried so they'll still be blocked in the same way
            // as before. We can fast bail out.
            return null;
          }
        }
      }
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderExpirationTime,
      );
    } else {
      // An update was scheduled on this fiber, but there are no new props
      // nor legacy context. Set this to false. If an update queue or context
      // consumer produces a changed value, it will set this to true. Otherwise,
      // the component will assume the children have not changed and bail out.
      didReceiveUpdate = false;
    }
  } else {
    didReceiveUpdate = false;
  }

  // Before entering the begin phase, clear pending update priority.
  // TODO: This assumes that we're about to evaluate the component and process
  // the update queue. However, there's an exception: SimpleMemoComponent
  // sometimes bails out later in the begin phase. This indicates that we should
  // move this assignment out of the common path and into each branch.

  workInProgress.expirationTime = NoWork;

  // 匹配当前Fiber对象的 tag 关于 Fiber tag 的具体值 上文有
  // 这里的每个 tag 匹配一个函数 分别处理不同的对象
  switch (workInProgress.tag) {
    case IndeterminateComponent: {
      return mountIndeterminateComponent(
        current,
        workInProgress,
        workInProgress.type,
        renderExpirationTime,
      );
    }
    case LazyComponent: {
      const elementType = workInProgress.elementType;
      return mountLazyComponent(
        current,
        workInProgress,
        elementType,
        updateExpirationTime,
        renderExpirationTime,
      );
    }
    case FunctionComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps);
      return updateFunctionComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderExpirationTime,
      );
    }
    case ClassComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps);
      return updateClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderExpirationTime,
      );
    }
    case HostRoot:
      return updateHostRoot(current, workInProgress, renderExpirationTime);
    case HostComponent:
      return updateHostComponent(current, workInProgress, renderExpirationTime);
    case HostText:
      return updateHostText(current, workInProgress);
    case SuspenseComponent:
      return updateSuspenseComponent(
        current,
        workInProgress,
        renderExpirationTime,
      );
    case HostPortal:
      return updatePortalComponent(
        current,
        workInProgress,
        renderExpirationTime,
      );
    case ForwardRef: {
      const type = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =
        workInProgress.elementType === type
          ? unresolvedProps
          : resolveDefaultProps(type, unresolvedProps);
      return updateForwardRef(
        current,
        workInProgress,
        type,
        resolvedProps,
        renderExpirationTime,
      );
    }
    case Fragment:
      return updateFragment(current, workInProgress, renderExpirationTime);
    case Mode:
      return updateMode(current, workInProgress, renderExpirationTime);
    case Profiler:
      return updateProfiler(current, workInProgress, renderExpirationTime);
    case ContextProvider:
      return updateContextProvider(
        current,
        workInProgress,
        renderExpirationTime,
      );
    case ContextConsumer:
      return updateContextConsumer(
        current,
        workInProgress,
        renderExpirationTime,
      );
    case MemoComponent: {
      const type = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      // Resolve outer props first, then resolve inner props.
      let resolvedProps = resolveDefaultProps(type, unresolvedProps);
      if (__DEV__) {
        if (workInProgress.type !== workInProgress.elementType) {
          const outerPropTypes = type.propTypes;
          if (outerPropTypes) {
            checkPropTypes(
              outerPropTypes,
              resolvedProps, // Resolved for outer only
              'prop',
              getComponentName(type),
            );
          }
        }
      }
      resolvedProps = resolveDefaultProps(type.type, resolvedProps);
      return updateMemoComponent(
        current,
        workInProgress,
        type,
        resolvedProps,
        updateExpirationTime,
        renderExpirationTime,
      );
    }
    case SimpleMemoComponent: {
      return updateSimpleMemoComponent(
        current,
        workInProgress,
        workInProgress.type,
        workInProgress.pendingProps,
        updateExpirationTime,
        renderExpirationTime,
      );
    }
    case IncompleteClassComponent: {
      const Component = workInProgress.type;
      const unresolvedProps = workInProgress.pendingProps;
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps);
      return mountIncompleteClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderExpirationTime,
      );
    }
    case SuspenseListComponent: {
      return updateSuspenseListComponent(
        current,
        workInProgress,
        renderExpirationTime,
      );
    }
    case FundamentalComponent: {
      if (enableFundamentalAPI) {
        return updateFundamentalComponent(
          current,
          workInProgress,
          renderExpirationTime,
        );
      }
      break;
    }
    case ScopeComponent: {
      if (enableScopeAPI) {
        return updateScopeComponent(
          current,
          workInProgress,
          renderExpirationTime,
        );
      }
      break;
    }
    case Block: {
      if (enableBlocksAPI) {
        const block = workInProgress.type;
        const props = workInProgress.pendingProps;
        return updateBlock(
          current,
          workInProgress,
          block,
          props,
          renderExpirationTime,
        );
      }
      break;
    }
  }
  invariant(
    false,
    'Unknown unit of work tag (%s). This error is likely caused by a bug in ' +
      'React. Please file an issue.',
    workInProgress.tag,
  );
}


```

## commit

### finishSyncRender

```ts
function finishSyncRender(root) {
  // Set this to null to indicate there's no in-progress render.
  // 将此值设置为 null 表示当前没有处于 render 中
  workInProgressRoot = null;
  commitRoot(root);
}
```

### commitRootImpl

```ts
function commitRootImpl(root, renderPriorityLevel) {
  // 省略部分代码
  // 函数拿到 root 上的 inishedWork 之后并将它设置为null
  // 省略部分代码
  do {
    {
      invokeGuardedCallback(null, commitBeforeMutationEffects, null);

      if (hasCaughtError()) {
        if (!(nextEffect !== null)) {
          {
            throw Error("Should be working on an effect.");
          }
        }

        var error = clearCaughtError();
        captureCommitPhaseError(nextEffect, error);
        nextEffect = nextEffect.nextEffect;
      }
    }
  } while (nextEffect !== null);
}
```

```ts
function insertOrAppendPlacementNodeIntoContainer(node, before, parent) {
  var tag = node.tag;
  var isHost = tag === HostComponent || tag === HostText;

  if (isHost || enableFundamentalAPI) {
    var stateNode = isHost ? node.stateNode : node.stateNode.instance;

    if (before) {
      insertInContainerBefore(parent, stateNode, before);
    } else {
      appendChildToContainer(parent, stateNode);
    }
  } else if (tag === HostPortal);
  else {
    var child = node.child;

    if (child !== null) {
      insertOrAppendPlacementNodeIntoContainer(child, before, parent);
      var sibling = child.sibling;

      while (sibling !== null) {
        insertOrAppendPlacementNodeIntoContainer(sibling, before, parent);
        sibling = sibling.sibling;
      }
    }
  }
}
```
