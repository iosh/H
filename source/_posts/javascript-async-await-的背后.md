---
title: javascript async await 的背后
date: 2022-04-14 15:39:43
tags:
---

便捷的 async await 背后是什么

<!-- more -->

打开 babel 的官网, 在 try it out 中输入

```JavaScript
async function a(){
	return await b()
}

async function b(){
	return 1
}
```

得到输出

```JavaScript
function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
    try {
        var info = gen[key](arg);
        var value = info.value;
    } catch (error) {
        reject(error);
        return;
    }
    if (info.done) {
        resolve(value);
    } else {
        Promise.resolve(value).then(_next, _throw);
    }
}

function _asyncToGenerator(fn) {
    return function() {
        var self = this,
            args = arguments;
        return new Promise(function(resolve, reject) {
            var gen = fn.apply(self, args);

            function _next(value) {
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value);
            }

            function _throw(err) {
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err);
            }
            _next(undefined);
        });
    };
}

function a() {
    return _a.apply(this, arguments);
}

function _a() {
    _a = _asyncToGenerator( /*#__PURE__*/ regeneratorRuntime.mark(function _callee() {
        return regeneratorRuntime.wrap(function _callee$(_context) {
            while (1) {
                switch (_context.prev = _context.next) {
                    case 0:
                        _context.next = 2;
                        return b();

                    case 2:
                        return _context.abrupt("return", _context.sent);

                    case 3:
                    case "end":
                        return _context.stop();
                }
            }
        }, _callee);
    }));
    return _a.apply(this, arguments);
}

```

可以看到 a b 两个函数都生成了一个同名带下划线的函数.

再继续下去之前, 需要看一下 JavaScript 历史, 在 ES6 中新增了 Promise 终结了回调函数的噩梦. Generators 则添加了以一个神奇的特性, 允许函数保留状态暂时交出控制权.

一般而言一个函数的执行是从上到下的, 直至执行完成整个函数

```JavaScript
// a 函数内的整个函数体都会被执行.在此期间占用主线程, 直至结束
function a(){
    //do something
}
a()
```

而 generator 有些不同, 下面是一个 MDN 上的例子

```JavaScript
function* idMaker(){
    let index = 0;
    while(true)
        yield index++;
}

let gen = idMaker(); // "Generator { }"

console.log(gen.next().value);
// 0
console.log(gen.next().value);
// 1
console.log(gen.next().value);
// 2
// ...
```

generator 和普通函数声明方式不同, 可以看到 idMaker 有一个 while true.

在一般函数内这就是死循环. 但是 generator 每执行一次 while 会将控制权让出去, 并不占用主线程. 同时它又保留自己内部的状态, next 方法被调用之后又会继续执行.

async 是一个语法糖, 基于 Promise 和 generator 的一个语法糖, 明白这点之后, 继续回头看.

经过 babel 的转换, 在结果中可以看到多了三个新的东西 `_asyncToGenerator` `asyncGeneratorStep` `regeneratorRuntime`

## \_asyncToGenerator

从函数字面意思可以看到吧 async 转换为 generator, 他的参数是 `regeneratorRuntime` 这个最后再看.

```JavaScript
function _asyncToGenerator(fn) {
    return function() {
        var self = this,
            args = arguments;
        return new Promise(function(resolve, reject) {
            var gen = fn.apply(self, args);

            function _next(value) {
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value);
            }

            function _throw(err) {
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err);
            }
            _next(undefined);
        });
    };
}
```

可以看到 \_asyncToGenerator 返回一个 Promise , 这也是标准上说的 `async函数一定会返回一个promise对象。如果一个async函数的返回值看起来不是promise，那么它将会被隐式地包装在一个promise中。--- MDN`

函数内部 fn (regeneratorRuntime) 被调用并返回一个 generator . 并且声明了两个函数 \_next 和 \_throw 都是调用了 `asyncGeneratorStep` 参数分别传入了不同的值 `next` `throw`

## asyncGeneratorStep

同样的可以看到 asyncGeneratorStep 函数也很简单, 如果 generator 返回 done 那么直接 resolve Promise , 如果不是那么继续返回一个 Promise 参数 又是自己.

```JavaScript
function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
    try {
        /**
        *  根据参数不同, 调用的方法不同
        *  var info = gen['next'](arg)
        *  var info = gen['throw'](arg)
        */
        var info = gen[key](arg);
        var value = info.value;
    } catch (error) {
        reject(error);
        return;
    }
    if (info.done) {
        resolve(value);
    } else {
        Promise.resolve(value).then(_next, _throw);
    }
}

```

## regeneratorRuntime

[regeneratorRuntime 源代码](https://github.com/facebook/regenerator/blob/0566ce53c67b5fe1de13b47ed4db00570de7d9f6/packages/regenerator-runtime/runtime.js)

从下面的代码可以看到 `_asyncToGenerator` 的参数是 `regeneratorRuntime.mark` 而 mark 的参数 \_callee 直接返回 `regeneratorRuntime.wrap`

```JavaScript
_asyncToGenerator( /*#__PURE__*/ regeneratorRuntime.mark(function _callee() {
        return regeneratorRuntime.wrap(function _callee$(_context) {
            while (1) {
                switch (_context.prev = _context.next) {
                    case 0:
                        _context.next = 2;
                        return b();

                    case 2:
                        return _context.abrupt("return", _context.sent);

                    case 3:
                    case "end":
                        return _context.stop();
                }
            }
        }, _callee);
    }));

```

### regeneratorRuntime.mark

```javascript
exports.mark = function (genFun) {
  // 这里的 genFun 是一个函数 函数返回  regeneratorRuntime.wrap
  // 下面是设定原型
  if (Object.setPrototypeOf) {
    // 如果 genFun 是一个 对象那么就指定原型, 如果prototype参数不是一个对象或者null(例如，数字，字符串，boolean，或者 undefined)，则什么都不做
    Object.setPrototypeOf(genFun, GeneratorFunctionPrototype);
  } else {
    // 上面的替代方法
    genFun.__proto__ = GeneratorFunctionPrototype;
    define(genFun, toStringTagSymbol, "GeneratorFunction");
  }
  // 为方法指定原型
  genFun.prototype = Object.create(Gp);
  return genFun;
};
```

## regeneratorRuntime.wrap

看一下上面的 warp 是如何调用的

```javascript
_asyncToGenerator(
  /*#__PURE__*/ regeneratorRuntime.mark(function _callee() {
    return regeneratorRuntime.wrap(function _callee$(_context) {
      while (1) {
        switch ((_context.prev = _context.next)) {
          case 0:
            _context.next = 2;
            return b();

          case 2:
            return _context.abrupt("return", _context.sent);

          case 3:
          case "end":
            return _context.stop();
        }
      }
    }, _callee);
  })
);
```

```JavaScript
    // 这里的 innerFn 是一个 _callee$ 函数, 内部是一个 while(1) 的循环
    // 这里的 outerFn 是被 mark 函数设定了原型的函数, 实际上也就是 调用 warp 的函数

    function wrap(innerFn, outerFn, self, tryLocsList) {
        // 如果提供了 otherFn并且 prototype 是 Generator 那么就使用 outerFn  否则使用 Generator
        // If outerFn provided and outerFn.prototype is a Generator, then outerFn.prototype instanceof Generator.
        var protoGenerator = outerFn && outerFn.prototype instanceof Generator ? outerFn : Generator;

        // 创建 Generator 对象
        var generator = Object.create(protoGenerator.prototype);
        // 创建 context
        var context = new Context(tryLocsList || []);

        // _invoke 方法实现了统一的 .next .throw 和 .return 方法
        // The ._invoke method unifies the implementations of the .next,
        // .throw, and .return methods.
        generator._invoke = makeInvokeMethod(innerFn, self, context);

        return generator;
    }
```

## \_next

之后回过头

```JavaScript
function _asyncToGenerator(fn) {
    return function() {
        var self = this,
            args = arguments;
        return new Promise(function(resolve, reject) {
            var gen = fn.apply(self, args);

            function _next(value) {
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value);
            }

            function _throw(err) {
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err);
            }
            _next(undefined);
        });
    };
}
```

这里调用了 \_next 来执行函数

```JavaScript
function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
    try {
        // 这里传入的 key 是next 也就是读取 generator 对象的 next 方法
        // 最后调用的是 _invoke(method)
        var info = gen[key](arg);
        var value = info.value;
    } catch (error) {
        reject(error);
        return;
    }
    // 这里判断是否完毕
    if (info.done) {
        resolve(value);
    } else {
        // 如果没有完毕那么继续 then
        Promise.resolve(value).then(_next, _throw);
    }
}

```

## Generator

使用 babel 转换如下代码

```JavaScript
function * a() {
  yield 1
  yield 2
  yield 3
  yield 4
}

```

得到结果

```JavaScript
"use strict";

var _marked = /*#__PURE__*/regeneratorRuntime.mark(a);

function a() {
  return regeneratorRuntime.wrap(function a$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          _context.next = 2;
          return 1;

        case 2:
          _context.next = 4;
          return 2;

        case 4:
          _context.next = 6;
          return 3;

        case 6:
          _context.next = 8;
          return 4;

        case 8:
        case "end":
          return _context.stop();
      }
    }
  }, _marked);
}
```

可以看到和 async 相比少了两个辅助函数.

```JavaScript
const b = a()
b.next() // {value: 1, done: false}
b.next() // {value: 2, done: false}
b.next() // {value: 3, done: false}
b.next() // {value: 4, done: false}
b.next() // {value: undefined, done: true}
```

这里每次的 next 执行实际上执行的都是 Generator 的 \_invoke 方法, `_invoke(method, arg)` `_invoke('next', arg)`

```javascript
function wrap(innerFn, outerFn, self, tryLocsList) {
  // If outerFn provided and outerFn.prototype is a Generator, then outerFn.prototype instanceof Generator.
  var protoGenerator =
    outerFn && outerFn.prototype instanceof Generator ? outerFn : Generator;
  var generator = Object.create(protoGenerator.prototype);
  var context = new Context(tryLocsList || []);

  // The ._invoke method unifies the implementations of the .next,
  // .throw, and .return methods.
  generator._invoke = makeInvokeMethod(innerFn, self, context);

  return generator;
}
```

wrap 方法提供的

```JavaScript
function makeInvokeMethod(innerFn, self, context) {

    /**
     * 这是定义的一些状态
     * var GenStateSuspendedStart = "suspendedStart";
     * var GenStateSuspendedYield = "suspendedYield";
     * var GenStateExecuting = "executing";
     * var GenStateCompleted = "completed";
    */
       // 这里定义自身状态
        var state = GenStateSuspendedStart;
        // 闭包, 返回 invoke 函数
        return function invoke(method, arg) {
            if (state === GenStateExecuting) {
                throw new Error("Generator is already running");
            }

            if (state === GenStateCompleted) {
                if (method === "throw") {
                    throw arg;
                }

                // Be forgiving, per 25.3.3.3.3 of the spec:
                // https://people.mozilla.org/~jorendorff/es6-draft.html#sec-generatorresume
                return doneResult();
            }

            context.method = method;
            context.arg = arg;

            while (true) {
                var delegate = context.delegate;
                if (delegate) {
                    var delegateResult = maybeInvokeDelegate(delegate, context);
                    if (delegateResult) {
                        if (delegateResult === ContinueSentinel) continue;
                        return delegateResult;
                    }
                }

                if (context.method === "next") {
                    // Setting context._sent for legacy support of Babel's
                    // function.sent implementation.
                    context.sent = context._sent = context.arg;

                } else if (context.method === "throw") {
                    if (state === GenStateSuspendedStart) {
                        state = GenStateCompleted;
                        throw context.arg;
                    }

                    context.dispatchException(context.arg);

                } else if (context.method === "return") {
                    context.abrupt("return", context.arg);
                }

                state = GenStateExecuting;
                // 尝试捕捉错误
                var record = tryCatch(innerFn, self, context);
                // 如果没有错误
                if (record.type === "normal") {
                    // If an exception is thrown from innerFn, we leave state ===
                    // GenStateExecuting and loop back for another invocation.
                    state = context.done
                        ? GenStateCompleted
                        : GenStateSuspendedYield;

                    if (record.arg === ContinueSentinel) {
                        continue;
                    }

                    return {
                        value: record.arg,
                        done: context.done
                    };

                } else if (record.type === "throw") {
                    state = GenStateCompleted;
                    // Dispatch the exception by looping back around to the
                    // context.dispatchException(context.arg) call above.
                    context.method = "throw";
                    context.arg = record.arg;
                }
            }
        };
    }

```

可以看到, next 调用之后每次都会调用 babel 写的 switch , 拿到返回值, 返回. 通过维护一个 state 和一个 context, 形成一个状态机.

```JavaScript
"use strict";

var _marked = /*#__PURE__*/regeneratorRuntime.mark(a);

function a() {
  return regeneratorRuntime.wrap(function a$(_context) {
      // next 函数被调用后, 每次这里也会被调用, 状态都存在 context 上.
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
            // 第一次调用, 修改 next 为2 下次调用的时候就会 匹配到 case 2
          _context.next = 2;
          return 1;

        case 2:
            // 第二次调用继续修改 next 下次就会是 4
          _context.next = 4;
          return 2;

        case 4:
          _context.next = 6;
          return 3;

        case 6:
          _context.next = 8;
          return 4;

        case 8:
            // 最终调用终止
        case "end":
          return _context.stop();
      }
    }
  }, _marked);
}

```

## 总结

async 会被 babel 首先转化为 Promise, 然后创建一个 Generator, 并且先调用一次 next 来获取结果.

普通的 Generator 就没有辅助函数, 单纯使用 regeneratorRuntime 来进行执行, babel 会拆解每个步骤作为 switch 的一个 case, regeneratorRuntime 实现标准的 next throw 等方法, 自身维护相关状态, 调用回调函数获取返回值.

