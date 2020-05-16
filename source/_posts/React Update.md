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

}
```
