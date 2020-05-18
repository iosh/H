---
title: React Update
date: 2020-05-16 18:38:39
tags: React
---

React DOM 中 React 是如何更新的

<!-- more -->

# class 是如何更新到的

在 React 中更新是通过调用 setState 来完成的,并且不保证是同步函数,研究一下.

React 部分

```tsx
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}

Component.prototype.setState = function (partialState, callback) {
  invariant(
    typeof partialState === "object" ||
      typeof partialState === "function" ||
      partialState == null,
    "setState(...): takes an object of state variables to update or a " +
      "function which returns an object of state variables."
  );
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};
```

在 React 中并为指定实际的 updater 对象, 而 React 本身并不提供这部分代码逻辑,通过搜索

可以在 React-DOM 中发现, 这一步是在初始化 Fiber 链表的时候对 class 进行实例化.

```ts
function constructClassInstance(workInProgress: Fiber, ctor: any, props: any) {
  // 省略代码
  const instance = new ctor(props, context);
  const state = (workInProgress.memoizedState =
    instance.state !== null && instance.state !== undefined
      ? instance.state
      : null);
  adoptClassInstance(workInProgress, instance);
  // 省略代码
}
function adoptClassInstance(workInProgress: Fiber, instance: any): void {
  instance.updater = classComponentUpdater;
  workInProgress.stateNode = instance;
  // The instance needs access to the fiber so that it can schedule updates
  // 实例需要能获得 fiber 对象才能进行更新
  // setInstance 给实例添加了一个属性.值是当前 Fiber 对象的引用
  setInstance(instance, workInProgress);
  if (__DEV__) {
    instance._reactInternalInstance = fakeInternalInstance;
  }
}
```

## classComponentUpdater

模块位于 react-reconciler 模块内,此模块负责 React 的更新调度

```ts

const classComponentUpdater = {
  isMounted, // 这个已经弃用了,用来表示当前组件是否卸载,应该是 class 以前的时代的 API
    // this.setState
  enqueueSetState(inst, payload, callback) {
    const fiber = getInstance(inst);
    const currentTime = requestCurrentTimeForUpdate();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig,
    );

    const update = createUpdate(expirationTime, suspenseConfig);

    update.payload = payload;
    if (callback !== undefined && callback !== null) {
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'setState');
      }
      update.callback = callback;
    }

    enqueueUpdate(fiber, update);
    scheduleUpdateOnFiber(fiber, expirationTime);

    if (__DEV__) {
      if (enableDebugTracing) {
        if (fiber.mode & DebugTracingMode) {
          const label = priorityLevelToLabel(
            ((update.priority: any): ReactPriorityLevel),
          );
          const name = getComponentName(fiber.type) || 'Unknown';
          logStateUpdateScheduled(name, label, payload);
        }
      }
    }
  },

  // 通过搜索发在在几个固定生命周期中调用
  enqueueReplaceState(inst, payload, callback) {
    const fiber = getInstance(inst);
    const currentTime = requestCurrentTimeForUpdate();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig,
    );

    const update = createUpdate(expirationTime, suspenseConfig);
    update.tag = ReplaceState;
    update.payload = payload;

    if (callback !== undefined && callback !== null) {
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'replaceState');
      }
      update.callback = callback;
    }

    enqueueUpdate(fiber, update);
    scheduleUpdateOnFiber(fiber, expirationTime);

    if (__DEV__) {
      if (enableDebugTracing) {
        if (fiber.mode & DebugTracingMode) {
          const label = priorityLevelToLabel(
            ((update.priority: any): ReactPriorityLevel),
          );
          const name = getComponentName(fiber.type) || 'Unknown';
          logStateUpdateScheduled(name, label, payload);
        }
      }
    }
  },

  // this.forceUpdate 强制更新
  enqueueForceUpdate(inst, callback) {
    const fiber = getInstance(inst);
    const currentTime = requestCurrentTimeForUpdate();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig,
    );

    const update = createUpdate(expirationTime, suspenseConfig);
    update.tag = ForceUpdate;

    if (callback !== undefined && callback !== null) {
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'forceUpdate');
      }
      update.callback = callback;
    }

    enqueueUpdate(fiber, update);
    scheduleUpdateOnFiber(fiber, expirationTime);

    if (__DEV__) {
      if (enableDebugTracing) {
        if (fiber.mode & DebugTracingMode) {
          const label = priorityLevelToLabel(
            ((update.priority: any): ReactPriorityLevel),
          );
          const name = getComponentName(fiber.type) || 'Unknown';
          logForceUpdateScheduled(name, label);
        }
      }
    }
  },
};

```

### enqueueSetState

```ts
{
    // 可以从上面看到 函数参数传递了四个参数最后一个参数是字符串 setState
  // 第一个参数是实例, 第二个是要被合并的对象或者函数,第三个就是回调函数可选
    enqueueSetState(inst, payload, callback) {
   //上面说到了 classComponentUpdater 被添加到实例的时候会给实例添加当前的fiber对象作为实例属性
    const fiber = getInstance(inst);


    // 计算一些时间
    const currentTime = requestCurrentTimeForUpdate();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig,
    );


   // 构建 update 形式如下
   /*
      update = {
        expirationTime: expirationTime,
        suspenseConfig: suspenseConfig,
        tag: UpdateState,
        payload: null,
        callback: null,
        next: null
      };
   */
    const update = createUpdate(expirationTime, suspenseConfig);

    // 将返回的 state Object  or  function 赋值供后续调用
    update.payload = payload;
    if (callback !== undefined && callback !== null) {
       // 这里是个开发判断, 判断如果存在callback 那么函数内部判断是不是一个函数或者 null, 如果都不是那么就报错提示用户正确使用方法
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'setState');
      }
      update.callback = callback;
    }
    // enqueueUpdate
    // 这个函数会读取 Fiber 对象上的 updateQueue 对象,判断 updateQueue.shared.padding 是否已经存在等待 上方生成的 update对象
    // 如果存在那么 会将next 指针指向这个新增的,如果不存在那么就赋值为这个新增的 update , 形式是一个单向链表
    // update.next -> newUpdate.next -> 如此的一个链表形式的对象.
    // Fiber.updateQueue.padding = update

    enqueueUpdate(fiber, update);
    // 安排更新
    scheduleUpdateOnFiber(fiber, expirationTime);

    if (__DEV__) {
      if (enableDebugTracing) {
        if (fiber.mode & DebugTracingMode) {
          const label = priorityLevelToLabel(
            ((update.priority: any): ReactPriorityLevel),
          );
          const name = getComponentName(fiber.type) || 'Unknown';
          logStateUpdateScheduled(name, label, payload);
        }
      }
    }
  },

}
```

### scheduleUpdateOnFiber

```ts
export function scheduleUpdateOnFiber(
  fiber: Fiber,
  expirationTime: ExpirationTime
) {
  // 这里会检查 nestedUpdateCount 的值是否大于阈值(50),如果大于就报错超过最大深度, 而nestedUpdateCount这个值
  // 会在 commitRootImpl 函数内 有条件更新
  checkForNestedUpdates();
  // 判断是否在 render 阶段进行更新,
  warnAboutRenderPhaseUpdatesInDEV(fiber);
  // 更新过期时间,并从当前节点返回根节点
  const root = markUpdateTimeFromFiberToRoot(fiber, expirationTime);
  if (root === null) {
    warnAboutUpdateOnUnmountedFiberInDEV(fiber);
    return;
  }

  // TODO: computeExpirationForFiber also reads the priority. Pass the
  // priority as an argument to that function and this one.
  // 获得优先级, 立即更新优先级最高, 然后是用户要求更新, 然后是普通更新等等, 调用setState属于用户要求更新
  const priorityLevel = getCurrentPriorityLevel();

  if (expirationTime === Sync) {
    if (
      // Check if we're inside unbatchedUpdates
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      // Check if we're not already rendering
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      // Register pending interactions on the root to avoid losing traced interaction data.
      schedulePendingInteractions(root, expirationTime);

      // This is a legacy edge case. The initial mount of a ReactDOM.render-ed
      // root inside of batchedUpdates should be synchronous, but layout updates
      // should be deferred until the end of the batch.
      performSyncWorkOnRoot(root);
    } else {
      // 这里来安排一个回调事件供调度器安排
      ensureRootIsScheduled(root);
      schedulePendingInteractions(root, expirationTime);
      // 如果以下判断成立那么这次的 setState 将是同步的否则是异步的
      if (executionContext === NoContext) {
        // Flush the synchronous work now, unless we're already working or inside
        // a batch. This is intentionally inside scheduleUpdateOnFiber instead of
        // scheduleCallbackForFiber to preserve the ability to schedule a callback
        // without immediately flushing it. We only do this for user-initiated
        // updates, to preserve historical behavior of legacy mode.
        // 立即进行同步工作, 除非我们已经在工作中,或者在批量处理中, 这是有意放在 scheduleUpdateOnFiber 中的,
        // 而不是 scheduleCallbackForFiber 以保留在不理解刷新回调的情况下安排调度的能力,只针对用户更新
        flushSyncCallbackQueue();
      }
    }
  } else {
    // Schedule a discrete update but only if it's not Sync.
    // 调度不相关的更新,但是值在非同步下
    if (
      (executionContext & DiscreteEventContext) !== NoContext &&
      // Only updates at user-blocking priority or greater are considered
      // discrete, even inside a discrete event.
      // 只有用户堵塞由下级更高的更新才被认为是布线管的, 即使在不相关事件内也是如此
      (priorityLevel === UserBlockingPriority ||
        priorityLevel === ImmediatePriority)
    ) {
      // This is the result of a discrete event. Track the lowest priority
      // discrete update per root so we can flush them early, if needed.
      // 这是不相关事件的结果, 追中根上的不相关的低优先级更新,这样就可以尽早的flush 他们, 如果需要
      if (rootsWithPendingDiscreteUpdates === null) {
        rootsWithPendingDiscreteUpdates = new Map([[root, expirationTime]]);
      } else {
        const lastDiscreteTime = rootsWithPendingDiscreteUpdates.get(root);
        if (
          lastDiscreteTime === undefined ||
          lastDiscreteTime > expirationTime
        ) {
          rootsWithPendingDiscreteUpdates.set(root, expirationTime);
        }
      }
    }
    // Schedule other updates after in case the callback is sync.
    // 安排其他更新防止同步回调
    ensureRootIsScheduled(root);
    schedulePendingInteractions(root, expirationTime);
  }
}
```

### ensureRootIsScheduled

```ts
// Use this function to schedule a task for a root. There's only one task per
// root; if a task was already scheduled, we'll check to make sure the
// expiration time of the existing task is the same as the expiration time of
// the next level that the root has work on. This function is called on every
// update, and right before exiting a task.
// 使用这个函数从 root 上安排任务, 每个 root 只有一个任务, 如果一个任务已经被安排,
// 我们将检查以确保现有的任务的过期时间,与root要处理的下一层任务的过期时间相同,
// 这个函数调用每个 update. 在完成任务之前.
function ensureRootIsScheduled(root: FiberRoot) {
  //从 root 上获取最后超时时间
  const lastExpiredTime = root.lastExpiredTime;
  if (lastExpiredTime !== NoWork) {
    // Special case: Expired work should flush synchronously.
    // 特殊条件下: 过期的任务应该被同步处理
    root.callbackExpirationTime = Sync;
    root.callbackPriority_old = ImmediatePriority;

    root.callbackNode = scheduleSyncCallback(
      performSyncWorkOnRoot.bind(null, root)
    );
    return;
  }

  const expirationTime = getNextRootExpirationTimeToWorkOn(root);
  const existingCallbackNode = root.callbackNode;
  if (expirationTime === NoWork) {
    // There's nothing to work on.
    // 没有什么工作
    if (existingCallbackNode !== null) {
      root.callbackNode = null;
      root.callbackExpirationTime = NoWork;
      root.callbackPriority_old = NoPriority;
    }
    return;
  }

  // TODO: If this is an update, we already read the current time. Pass the
  // time as an argument.
  // TODO: 如果这是一个 update, 我们已经读取当前时间, 通过参数传递这个时间
  const currentTime = requestCurrentTimeForUpdate();
  const priorityLevel = inferPriorityFromExpirationTime(
    currentTime,
    expirationTime
  );

  // If there's an existing render task, confirm it has the correct priority and
  // expiration time. Otherwise, we'll cancel it and schedule a new one.
  if (existingCallbackNode !== null) {
    const existingCallbackPriority = root.callbackPriority_old;
    const existingCallbackExpirationTime = root.callbackExpirationTime;
    if (
      // Callback must have the exact same expiration time.
      existingCallbackExpirationTime === expirationTime &&
      // Callback must have greater or equal priority.
      existingCallbackPriority >= priorityLevel
    ) {
      // Existing callback is sufficient.
      return;
    }
    // Need to schedule a new task.
    // TODO: Instead of scheduling a new task, we should be able to change the
    // priority of the existing one.
    cancelCallback(existingCallbackNode);
  }

  root.callbackExpirationTime = expirationTime;
  root.callbackPriority_old = priorityLevel;

  let callbackNode;
  if (expirationTime === Sync) {
    // Sync React callbacks are scheduled on a special internal queue
    // 同步 React 回调 被 安排在一个特殊的队列里面
    //scheduleSyncCallback 函数会将 performSyncWorkOnRoot 放在一个队列里面,这个函数会在到期之前被 flushSyncCallbackQueue 调用
    // performSyncWorkOnRoot 函数是不经过调度器的同步任务入口点
    callbackNode = scheduleSyncCallback(performSyncWorkOnRoot.bind(null, root));
  } else if (disableSchedulerTimeoutBasedOnReactExpirationTime) {
    callbackNode = scheduleCallback(
      priorityLevel,
      performConcurrentWorkOnRoot.bind(null, root)
    );
  } else {
    callbackNode = scheduleCallback(
      priorityLevel,
      performConcurrentWorkOnRoot.bind(null, root),
      // Compute a task timeout based on the expiration time. This also affects
      // ordering because tasks are processed in timeout order.
      { timeout: expirationTimeToMs(expirationTime) - now() }
    );
  }

  root.callbackNode = callbackNode;
}
```

所以 setState 构造一个 update 对象给当前 fiber 对象, 最后由 flushSyncCallbackQueue 函数去更新.
关于同步异步的问题, react 通过 判断当前上下文的值 executionContext 的值,如果是 0 那么说明不在 react 中,那么就是同步更新的.
否则就是异步的

### enqueueForceUpdate

```ts
{
   enqueueForceUpdate(inst, callback) {
    //...
    const update = createUpdate(expirationTime, suspenseConfig);
    // 可以看到差别就是这个
    /**
    export const UpdateState = 0;
    export const ReplaceState = 1;
    export const ForceUpdate = 2;
    export const CaptureUpdate = 3;
    */
    update.tag = ForceUpdate;
    //...
    enqueueUpdate(fiber, update);
    scheduleUpdateOnFiber(fiber, expirationTime);
   //...
  },
}
```

### flushSyncCallbackQueue

```ts
export function flushSyncCallbackQueue() {
  // immediateQueueCallbackNode 会在  scheduleSyncCallback 被赋值

  /**
  函数返回了类似这样的一个对象
    newTask = {
    id: taskIdCounter++, // 自增数字
    callback,// callback 这里传入的是 flushSyncCallbackQueueImpl
    priorityLevel, // 任务等级
    startTime, // 会通过函数获得一个时间
    expirationTime, // 根据优先级获得一个超时时间,最高优先级超时时间是 -1  
    sortIndex: -1,
  }
  */
  if (immediateQueueCallbackNode !== null) {
    const node = immediateQueueCallbackNode;
    immediateQueueCallbackNode = null;
    Scheduler_cancelCallback(node); //  可以从下面看到这里取消掉了task 的callback
  }
  flushSyncCallbackQueueImpl();
}
```

```ts
function Scheduler_cancelCallback(task) {
  if (enableProfiling) {
    if (task.isQueued) {
      const currentTime = getCurrentTime();
      markTaskCanceled(task, currentTime);
      task.isQueued = false;
    }
  }

  // Null out the callback to indicate the task has been canceled. (Can't
  // remove from the queue because you can't remove arbitrary nodes from an
  // array based heap, only the first one.)
  task.callback = null;
}
```

### flushSyncCallbackQueueImpl

```ts
function flushSyncCallbackQueueImpl() {
  // 判断当前没有在 flush 同步队列以及队列不为空
  if (!isFlushingSyncQueue && syncQueue !== null) {
    // Prevent re-entrancy.
    // 对flag进行赋值,防止重复进行
    isFlushingSyncQueue = true;
    let i = 0;
    try {
      const isSync = true;
      const queue = syncQueue;
      // 这里开始运行队列任务
      runWithPriority(ImmediatePriority, () => {
        for (; i < queue.length; i++) {
          let callback = queue[i];
          do {
            // callback 会被转到  performSyncWorkOnRoot 函数这里调用 workLoopSync 进行整个流程 初次渲染流程在上一篇 ReactDOM中有
            //这里主要看一下 如何 diff code
            callback = callback(isSync);
          } while (callback !== null);
        }
      });
      syncQueue = null;
    } catch (error) {
      // If something throws, leave the remaining callbacks on the queue.
      if (syncQueue !== null) {
        syncQueue = syncQueue.slice(i + 1);
      }
      // Resume flushing in the next tick
      Scheduler_scheduleCallback(
        Scheduler_ImmediatePriority,
        flushSyncCallbackQueue
      );
      throw error;
    } finally {
      isFlushingSyncQueue = false;
    }
  }
}
```

workLoopConcurrent 开始调用 performUnitOfWork 来遍历链表, performUnitOfWork 开始调用 beginWork 开始处理每个节点

后面主要看如何比对新旧节点找出差异进行更新.

```ts
function performUnitOfWork(unitOfWork) {
// 两棵树是交替进行的,这里取 alternate 作为 current
var current = unitOfWork.alternate;
var next;
if ((unitOfWork.mode & ProfileMode) !== NoMode) {
  startProfilerTimer(unitOfWork);
  next = beginWork$1(current, unitOfWork, renderExpirationTime$1);
  //...
}

//...
```

```ts
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes
): Fiber | null {
  //...
  if (current !== null) {
    // 非第一次渲染 current 就不等 null 了

    // 新旧 props
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
      // 如果 props 或者 context 变化了, 将标记这个 fiber 完成了工作
      didReceiveUpdate = true;
    } else if (!includesSomeLane(renderLanes, updateLanes)) {
      //...
  } else {
    // 初次渲染 因为 alternate === null 所以current ===null 走 这里的逻辑
  }

    workInProgress.lanes = NoLanes;
  // 匹配标签处理
  switch (workInProgress.tag) {
    case IndeterminateComponent: {
      return mountIndeterminateComponent(
        current,
        workInProgress,
        workInProgress.type,
        renderLanes,
      );
    }
    case LazyComponent: {
      const elementType = workInProgress.elementType;
      return mountLazyComponent(
        current,
        workInProgress,
        elementType,
        updateLanes,
        renderLanes,
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
        renderLanes,
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
        renderLanes,
      );
    }
    case HostRoot:
      return updateHostRoot(current, workInProgress, renderLanes);
    case HostComponent:
      return updateHostComponent(current, workInProgress, renderLanes);
    case HostText:
      return updateHostText(current, workInProgress);
    case SuspenseComponent:
      return updateSuspenseComponent(current, workInProgress, renderLanes);
    case HostPortal:
      return updatePortalComponent(current, workInProgress, renderLanes);
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
        renderLanes,
      );
    }
    case Fragment:
      return updateFragment(current, workInProgress, renderLanes);
    case Mode:
      return updateMode(current, workInProgress, renderLanes);
    case Profiler:
      return updateProfiler(current, workInProgress, renderLanes);
    case ContextProvider:
      return updateContextProvider(current, workInProgress, renderLanes);
    case ContextConsumer:
      return updateContextConsumer(current, workInProgress, renderLanes);
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
        updateLanes,
        renderLanes,
      );
    }
    case SimpleMemoComponent: {
      return updateSimpleMemoComponent(
        current,
        workInProgress,
        workInProgress.type,
        workInProgress.pendingProps,
        updateLanes,
        renderLanes,
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
        renderLanes,
      );
    }
    case SuspenseListComponent: {
      return updateSuspenseListComponent(current, workInProgress, renderLanes);
    }
    case FundamentalComponent: {
      if (enableFundamentalAPI) {
        return updateFundamentalComponent(current, workInProgress, renderLanes);
      }
      break;
    }
    case ScopeComponent: {
      if (enableScopeAPI) {
        return updateScopeComponent(current, workInProgress, renderLanes);
      }
      break;
    }
    case Block: {
      if (enableBlocksAPI) {
        const block = workInProgress.type;
        const props = workInProgress.pendingProps;
        return updateBlock(current, workInProgress, block, props, renderLanes);
      }
      break;
    }
    case OffscreenComponent: {
      return updateOffscreenComponent(current, workInProgress, renderLanes);
    }
    case LegacyHiddenComponent: {
      return updateLegacyHiddenComponent(current, workInProgress, renderLanes);
    }
  }
}
```

下面就是实际更新当前值了

```ts
function commitWork(current: Fiber | null, finishedWork: Fiber): void {
  if (!supportsMutation) {
    switch (finishedWork.tag) {
      case FunctionComponent:
      case ForwardRef:
      case MemoComponent:
      case SimpleMemoComponent:
      case Block: {
        // Layout effects are destroyed during the mutation phase so that all
        // destroy functions for all fibers are called before any create functions.
        // This prevents sibling component effects from interfering with each other,
        // e.g. a destroy function in one component should never override a ref set
        // by a create function in another component during the same commit.
        if (
          enableProfilerTimer &&
          enableProfilerCommitHooks &&
          finishedWork.mode & ProfileMode
        ) {
          try {
            startLayoutEffectTimer();
            commitHookEffectListUnmount(
              HookLayout | HookHasEffect,
              finishedWork,
            );
          } finally {
            recordLayoutEffectDuration(finishedWork);
          }
        } else {
          commitHookEffectListUnmount(HookLayout | HookHasEffect, finishedWork);
        }
        return;
      }
      case Profiler: {
        return;
      }
      case SuspenseComponent: {
        commitSuspenseComponent(finishedWork);
        attachSuspenseRetryListeners(finishedWork);
        return;
      }
      case SuspenseListComponent: {
        attachSuspenseRetryListeners(finishedWork);
        return;
      }
      case HostRoot: {
        if (supportsHydration) {
          const root: FiberRoot = finishedWork.stateNode;
          if (root.hydrate) {
            // We've just hydrated. No need to hydrate again.
            root.hydrate = false;
            commitHydratedContainer(root.containerInfo);
          }
        }
        break;
      }
    }

    commitContainer(finishedWork);
    return;
  }

  switch (finishedWork.tag) {
    case FunctionComponent:
    case ForwardRef:
    case MemoComponent:
    case SimpleMemoComponent:
    case Block: {
      // Layout effects are destroyed during the mutation phase so that all
      // destroy functions for all fibers are called before any create functions.
      // This prevents sibling component effects from interfering with each other,
      // e.g. a destroy function in one component should never override a ref set
      // by a create function in another component during the same commit.
      if (
        enableProfilerTimer &&
        enableProfilerCommitHooks &&
        finishedWork.mode & ProfileMode
      ) {
        try {
          startLayoutEffectTimer();
          commitHookEffectListUnmount(HookLayout | HookHasEffect, finishedWork);
        } finally {
          recordLayoutEffectDuration(finishedWork);
        }
      } else {
        commitHookEffectListUnmount(HookLayout | HookHasEffect, finishedWork);
      }
      return;
    }
    case ClassComponent: {
      return;
    }
    case HostComponent: {
      const instance: Instance = finishedWork.stateNode;
      if (instance != null) {
        // Commit the work prepared earlier.
        const newProps = finishedWork.memoizedProps;
        // For hydration we reuse the update path but we treat the oldProps
        // as the newProps. The updatePayload will contain the real change in
        // this case.
        const oldProps = current !== null ? current.memoizedProps : newProps;
        const type = finishedWork.type;
        // TODO: Type the updateQueue to be specific to host components.
        const updatePayload: null | UpdatePayload = (finishedWork.updateQueue: any);
        finishedWork.updateQueue = null;
        if (updatePayload !== null) {
          commitUpdate(
            instance,
            updatePayload,
            type,
            oldProps,
            newProps,
            finishedWork,
          );
        }
        if (enableDeprecatedFlareAPI) {
          const prevListeners = oldProps.DEPRECATED_flareListeners;
          const nextListeners = newProps.DEPRECATED_flareListeners;
          if (prevListeners !== nextListeners) {
            updateDeprecatedEventListeners(nextListeners, finishedWork, null);
          }
        }
      }
      return;
    }
    case HostText: {
      invariant(
        finishedWork.stateNode !== null,
        'This should have a text node initialized. This error is likely ' +
          'caused by a bug in React. Please file an issue.',
      );
      const textInstance: TextInstance = finishedWork.stateNode;
      const newText: string = finishedWork.memoizedProps;
      // For hydration we reuse the update path but we treat the oldProps
      // as the newProps. The updatePayload will contain the real change in
      // this case.
      const oldText: string =
        current !== null ? current.memoizedProps : newText;
      commitTextUpdate(textInstance, oldText, newText);
      return;
    }
    case HostRoot: {
      if (supportsHydration) {
        const root: FiberRoot = finishedWork.stateNode;
        if (root.hydrate) {
          // We've just hydrated. No need to hydrate again.
          root.hydrate = false;
          commitHydratedContainer(root.containerInfo);
        }
      }
      return;
    }
    case Profiler: {
      return;
    }
    case SuspenseComponent: {
      commitSuspenseComponent(finishedWork);
      attachSuspenseRetryListeners(finishedWork);
      return;
    }
    case SuspenseListComponent: {
      attachSuspenseRetryListeners(finishedWork);
      return;
    }
    case IncompleteClassComponent: {
      return;
    }
    case FundamentalComponent: {
      if (enableFundamentalAPI) {
        const fundamentalInstance = finishedWork.stateNode;
        updateFundamentalComponent(fundamentalInstance);
        return;
      }
      break;
    }
    case ScopeComponent: {
      if (enableScopeAPI) {
        const scopeInstance = finishedWork.stateNode;
        if (enableDeprecatedFlareAPI) {
          const newProps = finishedWork.memoizedProps;
          const oldProps = current !== null ? current.memoizedProps : newProps;
          const prevListeners = oldProps.DEPRECATED_flareListeners;
          const nextListeners = newProps.DEPRECATED_flareListeners;
          if (prevListeners !== nextListeners || current === null) {
            updateDeprecatedEventListeners(nextListeners, finishedWork, null);
          }
        }
        prepareScopeUpdate(scopeInstance, finishedWork);
        return;
      }
      break;
    }
  }
  invariant(
    false,
    'This unit of work tag should not have side-effects. This error is ' +
      'likely caused by a bug in React. Please file an issue.',
  );
}

```


