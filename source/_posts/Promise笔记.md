---
title: Promise笔记
date: 2019-05-17 10:47:31
tags: javaScript
---

深入学习 JavaScript 中的 Promise.

<!-- more -->

本质上讲 Promise 就是一个对象, 它代表了一个异步操作的`最终`完成或者失败.

在使用 Promise 的时候,会有以下约定.

1. 在本轮 JavaScript event loop(事件循环)完成之前,callback 不会执行.
2. 因为 1 所以 通过 .then() 形式添加的回调函数总会被调用,几遍是在异步完成之后才被添加的函数
3. 通过多次调用 .then() 可以添加多个回调函数, 并且他们会按照插入顺序独立运行.

基于上面的约定可以推测以下代码

```js
console.log(1);
new Promise((resolve, reject) => resolve(2)).then(n => console.log(n));
console.log(3);
console.log(4);
console.log(5);
```

2 是会在 5 之后出现的, 代码执行到第二行之后 resolve 传递一个数字 2,之后被放在下一轮队列中进行等待, 等到 `console.log(5)` 执行完毕之后,本轮代码就执行完了,就开始执行下一轮代码,所以最后打印出 2

```js
console.log(1);
setTImeout(() => console.log(2), 0);
new Promise((resolve, reject) => resolve(3)).then(n => console.log(n));

console.log(4);
console.log(5);
console.log(6);
```

上面的代码中 `setTimeout` 是最后才输出的, 经过查证,是应为在 事件队列中 Promise 比 setTimeout 拥有更高的优先级

[es6-promise](https://github.com/stefanpenner/es6-promise) 是一个使用 ES6 语法的 Promise 的 polyfill

先写一个最简单的 Promise 使用 demo.

```js
const promisel = new Promise((resolve, reject) =>
  setTimeout(() => resolve("hell"), 300)
);
promisel.then(value => console.log(value));
```

根据上面的例子可以说明 Promise 是一个构造函数, 构造函数接受一个函数作为参数,这个函数参数会得到两个函数`resolve`和`reject`作为参数,之后在回调函数中执行自己想要的操作,最终调用`resolve`或者`reject`并传入需要传递的参数.

之后就可以通过.then 方法来得到想要的结果.

```js
// 初始构造函数

class Promise {
  constructor(resolver) {
    // 接受一个函数作为参数
    this[PROMISE_ID] = nextId(); // PROMISE_ID 是一个随机字符串, nextID 是一个自增的数字
    this._result = this._state = undefined; // 初始化结果和状态
    this._subscribers = []; // 订阅数组

    // 在 Promise.all 中 Enumerator 会调用 new Promise(noop),这里主要判断是不是内部调用如果是就不需要初始化和检查了.
    if (noop !== resolver) {
      // 判断参数是否为函数如果是函数则进行初始化
      typeof resolver !== "function" && needsResolver();
      // 判断 Promise 是否为 new Promise调用如果是则初始化,如果不是则抛出错误
      this instanceof Promise ? initializePromise(this, resolver) : needsNew();
    }
  }
}

/*
 * 初始化 Promise
 */

// 从上面得知 参数1 是Promise class 本身, resolver 是用户传入的回调函数
function initializePromise(promise, resolver) {
  try {
    // 给用户的回调传入 resolvePromise  和 rejectPromise
    resolver(
      function resolvePromise(value) {
        resolve(promise, value);
      },
      function rejectPromise(reason) {
        reject(promise, reason);
      }
    );
  } catch (e) {
    // 抛出错误
    reject(promise, e);
  }
}

function resolve(promise, value) {
  if (promise === value) {
    // 判断用户 resolve(value) 的value是否为 Promise 本身如果是 则直接 reject 并抛出错误
    reject(promise, selfFulfillment());
  } else if (objectOrFunction(value)) {
    // 判断参数是否为空
    handleMaybeThenable(promise, value, getThen(value)); // 函数处理用户传入的值, 这里要包括处理想 Promise.resolve() 这种情况.所以会判断 传入的值是否为 Promise 和 传入的对象是否携带 then 方法
  } else {
    fulfill(promise, value); // 如果参数为 null 则直接完成
  }
}

// 开始处理用户传入的对象
function handleMaybeThenable(promise, maybeThenable, then) {
  if (
    maybeThenable.constructor === promise.constructor && // 判断 用户传入的对象是不是传入 es6-Promise 中的 Promise
    then === originalThen && // 判断如果是 es6-Promise 那么在对比value 的then 方法是不是 es6-Promise 自带的
    maybeThenable.constructor.resolve === originalResolve
  ) {
    // 和上面的对比一样 判断 resolve是不是自带的
    handleOwnThenable(promise, maybeThenable); // 如果都相符(参数是Promise而且是es6-Promise本身) 调用函数处理 函数每部判断promise._state 的状态码分别进行 fulfill (完成) reject (拒绝) subscribe (订阅.实际上就是在等待等待成功或者失败订阅发布模式事件成功或者之后进行发布)
  } else {
    if (then === TRY_CATCH_ERROR) {
      // getThen 方法在读取 vlue.then 的时候如果报错会将错误值赋值给 TRY_CATCH_ERROR
      reject(promise, TRY_CATCH_ERROR.error); // 如果有错误则直接reject 并携带错误值
      TRY_CATCH_ERROR.error = null; // 清空临时储存错误的对象
    } else if (then === undefined) {
      // 如果 value 不存在 then方法
      fulfill(promise, maybeThenable); // 则立即成功
    } else if (isFunction(then)) {
      // 如果存在then方法 并且 then 方法是一个函数
      handleForeignThenable(promise, maybeThenable, then); // 处理原生的带 Promise方法
    } else {
      fulfill(promise, maybeThenable); // 都不符合则直接成功
    }
  }
}
```

到这里捋一下目前为止已知的东西,然后再继续往下往下看

```js
// 情况1
const a = new Promise((resolve, reject) => resolve(1));
```

执行步骤:

1. Promise 构造函数中检查 Promise 的参数, 并且调用 initializePromise 函数进行初始化 initializePromise 函数给参数传入 resolv 和 reject 两个函数

2. 在上面的示例中,最后调用了 resolve(1), resolve 函数对参数合法性进行检查包括参数不能使 Promise 本身, 之后调用 handleMaybeThenable 函数来处理参数

3. handleMaybeThenable 收到参数之后又是一顿检查,主要是判断参数是不是一个 Promise 如果是那么则判断内部的状态码,分别执行对应的操作,如果不是 Promise 那么直接 fulfill (成功)

那么接下来就是看 fulfill 函数和 reject 函数了

```js
function fulfill(promise, value) {
  if (promise._state !== PENDING) {
    // 判断当前状态是否为等待状态如果是那么不做处理
    return;
  }

  promise._result = value; // 如果当前状态不是等待中那么将值存起来
  promise._state = FULFILLED; // 将当前状态更新为成功

  if (promise._subscribers.length !== 0) {
    // 判断订阅数组长度(就是判断当前是否有还需要处理的事件如果有那么久调用处理, 订阅发布模型)
    asap(publish, promise); // asap 是 reply as soon as possible 尽快回复的意思
  }
}

function reject(promise, reason) {
  // reject 失败函数
  if (promise._state !== PENDING) {
    // 同样判断当前状态是否为等待中,如果是那么不处理
    return;
  }
  promise._state = REJECTED; // 更改当前状态为失败
  promise._result = reason; // 储存失败结果

  asap(publishRejection, promise); // 调用 asap
}
```

fulfill 和 reject 都涉及到一个 asap 函数, 和各自的 publish,publishRejection 那么接下来看看这个几个函数

```js
// subscribe 函数在 handleOwnThenable 函数内调用
// 首先用户使用的时候必须 resolve(promise) 也就是说 resolve 的参数必须是一个 promise 才会调用 subscribe

// 在  handleOwnThenable  中调用的形式 是
// subscribe(thenable, undefined, value  => resolve(promise, value),reason => reject(promise, reason))
function subscribe(parent, child, onFulfillment, onRejection) {
  let { _subscribers } = parent; // parent 使用户传入的 promise child 一般都是 undefined
  let { length } = _subscribers; // 读取当前列表长度

  parent._onerror = null; // 将错误赋值为空

  _subscribers[length] = child; // 赋值
  _subscribers[length + FULFILLED] = onFulfillment; // resolve 回调
  _subscribers[length + REJECTED] = onRejection; // reject 回调

  if (length === 0 && parent._state) {
    // 如果长度为0,或者状态值为成功或者失败 等待中的值为1
    asap(publish, parent);
  }
}

function publish(promise) {
  //  接受要处理的promise实例对象
  let subscribers = promise._subscribers; // 订阅数组
  let settled = promise._state; // 读取当前状态

  if (subscribers.length === 0) {
    return;
  } // 判断订阅数组长度是否为0 如果为0 则不需要处理 subscribers数组赋值在下面一点

  let child,
    callback,
    detail = promise._result; // 拿到用户储存的值

  for (let i = 0; i < subscribers.length; i += 3) {
    // 遍历
    child = subscribers[i]; // child 一般为 undefined
    callback = subscribers[i + settled];

    if (child) {
      invokeCallback(settled, child, callback, detail);
    } else {
      callback(detail);
    }
  }

  promise._subscribers.length = 0;
}

function publishRejection(promise) {
  if (promise._onerror) {
    promise._onerror(promise._result);
  }

  publish(promise);
}

export var asap = function asap(callback, arg) {
  queue[len] = callback; // queue 是一个数组,在这里模拟等待队列.队列是先进先出的, 而 len 是一个变量用于记录长度
  queue[len + 1] = arg; // 将当前的 Promise 存在队列里(入队)
  len += 2; // 一次时间占两个位置 第一个位置是回调第二个位置是参数
  if (len === 2) {
    // If len is 2, that means that we need to schedule an async flush.
    // If additional callbacks are queued before the queue is flushed, they
    // will be processed by this flush that we are scheduling.
    // 如果只有两个那么此次只需要处理当前事件即可,如果队列长度不止两个那么说明还有其他的事件等待处理 (ps 为什么用一个变量来做数组/
    //长度以为在源码中queue的值是Array(1000) 无法通过数组长度拿到实际队列长度)
    if (customSchedulerFn) {
      // 如果自定义调度器存在则使用自定义调度器如果不存在那么使用默认调度器
      customSchedulerFn(flush); // 使用 es6-promise 公开了一个Promise._setScheduler属性来设置自定义调度器
    } else {
      scheduleFlush(); // 使用默认调度器直接处理当前事件
    }
  }
};
```

scheduleFlush
事件调度器

```js
//这段if else 用来兼容多种运行环境

let scheduleFlush;
// Decide what async method to use to triggering processing of queued callbacks:
// 决定使用什么异步的方法来处理回调
if (isNode) {
  // 如果在node (isNode 是一连串判断判断一些全局变量是否存在)
  scheduleFlush = useNextTick(); // process.nextTick(flush)
} else if (BrowserMutationObserver) {
  scheduleFlush = useMutationObserver(); // 使用 BrowserMutationObserver 这个浏览器 api
} else if (isWorker) {
  scheduleFlush = useMessageChannel(); // 如果支持web worker 则使用 MessageChannel
} else if (browserWindow === undefined && typeof require === "function") {
  scheduleFlush = attemptVertx(); // 桌面程序
} else {
  scheduleFlush = useSetTimeout(); // 最终办法定时器.
}
```

这里只看 useMutationObserver 和 useSetTimeout 好了其他的如果有兴趣可以看

```js
function useMutationObserver() {
  let iterations = 0;
  const observer = new BrowserMutationObserver(flush); // 这个api提供的监听 dom变化的, 这里添加将事件添加进去
  const node = document.createTextNode(""); // 创建一个文本节点
  observer.observe(node, { characterData: true }); // observe方法接受两个方法目标节点,和参数这里设置了characterData
  //characterData  设定为true则监听目标节点文字变化

  return () => {
    // 返回一个匿名函数 这个函数执行之后会改变节点,之后就会触发 BrowserMutationObserver 执行flush事件
    node.data = iterations = ++iterations % 2;
  };
}

// 这个比较简单了,没什么好解释的了
function useSetTimeout() {
  // Store setTimeout reference so es6-promise will be unaffected by
  // other code modifying setTimeout (like sinon.useFakeTimers())
  const globalSetTimeout = setTimeout;
  return () => globalSetTimeout(flush, 1);
}
```

事件调度处理器看完了那么需要看 flush 方法.看看这个方法究竟做了什么

```js
function flush() {
  for (let i = 0; i < len; i += 2) {
    // += 2 是为了跳过 poromise 元素 queue 的构成是这样的 [callback, promise]
    let callback = queue[i];
    let arg = queue[i + 1];

    callback(arg); //

    queue[i] = undefined;
    queue[i + 1] = undefined;
  }

  len = 0;
}
```
