---
title: React Hooks
date: 2020-05-19 11:21:47
tags: React
---

React Hooks

<!-- more -->

在 React 内 hooks 是这样定义的,以 useState 为例

```ts
export function useState<S>(
  initialState: (() => S) | S
): [S, Dispatch<BasicStateAction<S>>] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
```

其他几个 hooks 形式都差不多,都是调用 resolveDispatcher 返回一个对象,然后调用对象上的具体方法.

# resolveDispatcher

```ts
function resolveDispatcher() {
  // ReactCurrentDispatcher 是一个 export 出来的对象,current 会在合适的时候进行赋值
  const dispatcher = ReactCurrentDispatcher.current;

  // 下面这个报错应该都见过
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

通过翻阅源码可以发现对 ReactCurrentDispatcher.current 赋值的函数位于 packages/react-reconciler/src/ReactFiberHooks.old.js 的 renderWithHooks 函数, 而此函数被核心函数 beginWork 进行调用.


