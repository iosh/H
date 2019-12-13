---
title: Immer源码学习
date: 2018-07-22 12:27:24
tags: JavaScript
---

上周使用了 Immer 帮助我更新 redux state 对于萌新的我来说非常好用,粗略的看了一下源码,感觉很简单,尝试进行学习一下

<!-- more -->

# Immer 简介

Immer 说明文件中,简介明了的说明了自己的用处:

> 通过简单的修改当前的状态树来创建下一个不可变的状态树

Immer 暴露了一个可以完成所有工作的默认函数,函数签名如下:

`produce(currentState, producer: (draftState) => void): nextState`

同时它还有一个柯里化后的重载

这个库用起来非常简单

```javascript
import produce from "immer";

const baseState = [
  {
    todo: "Learn typescript",
    done: true
  },
  {
    todo: "Try immer",
    done: false
  }
];

const nextState = produce(baseState, draftState => {
  draftState.push({ todo: "Tweet about it" });
  draftState[1].done = true;
});
```

函数 produce 接收两个参数, 第一个参数是当前的状态树,第二个参数是一个函数,函数接受一个参数,可以对这个参数进行修改,直接赋值即可,最后 produce 函数返回的就是一个新的状态树(不需要 return)

从 Immer 库中开始看

# 柯里化准备和参数判断

```JavaScript

export {setAutoFreeze, setUseProxies} from "./common"

import {isProxyable, getUseProxies} from "./common"
import {produceProxy} from "./proxy"
import {produceEs5} from "./es5"

/**
 * produce 得到一个 state, 并且针对它运行一个函数.
 * 这个函数可以自由的改变这个 state, 因为它基于 copies-on-write(写入时赋值)的机制.
 * 这意味着原始状态将保持不变,一旦函数运行完毕,它将返回修改后的状态
 *
 * @export
 * @param {any} baseState - 原始的 state
 * @param {Function} producer - 一个函数接受一个基本 state 代理并且可以自由修改它的函数
 * @returns {any} 新的 state, 如果没有修改 state 的话就返回 baseState
 */
export default function produce(baseState, producer) {
    // 检查函数参数,预期得到1个或者两个参数,如果不是一个或者两个则抛出错误
    if (arguments.length !== 1 && arguments.length !== 2) throw new Error("produce expects 1 or 2 arguments, got " + arguments.length)

    // curried invocation 判断第一个函数是否为函数,如果是函数那么就是柯里化调用
    if (typeof baseState === "function") {
        // prettier-ignore 如果第一个参数是一个函数,那么就是柯里化调用,则第二个函数必定不能为函数(如果第一个参数为函数,第二个参数可以为数组或者对象,作为初始化状态)
        if (typeof producer === "function") throw new Error("if first argument is a function (curried invocation), the second argument to produce cannot be a function")

        const initialState = producer // 储存柯里化调用的初始化状态
        const recipe = baseState // 储存函数

        return function() { // 返回一个匿名函数
            const args = arguments // 声明一个变量储存参数

            const currentState = // 三元表达判断一下是否传入了第一个参数和第二个参数
                args[0] === undefined && initialState !== undefined
                    ? initialState // 如果第一个参数为未定义,第二个参数存在则返回第二个参数(即初始化状态)
                    : args[0] // 否则就返回第一个参数(上面已经判断了至少要有一个参数)

            return produce(currentState, draft => { // 上面收集到相关数据了,这里就进行调用
            // 转变为普通调用 produce(当前state, 可以修改的state)
                args[0] = draft // blegh!
                return recipe.apply(draft, args)
            })
        }
    }

    // prettier-ignore
    // 如果第一个参数不是函数,那么就不是柯里化,第二个参数必然是函数
    {
        if (typeof producer !== "function") throw new Error("if first argument is not a function, the second argument to produce should be a function")
    }

    // if state is a primitive, don't bother proxying at all
    // 如果 原始数据不是数组或者对象,或者为空就不需要代理了
    if (typeof baseState !== "object" || baseState === null) {
        const returnValue = producer(baseState)
        return returnValue === undefined ? baseState : returnValue
    }

    // 这里调用函数判断原始对象是否为代理对象(就判断他是否为原始JavaScript数组和对象)

    /**
     * isProxyable 接收一个接受一个值作为参数 判断值是否为原始数据类型
     * 其中判断了值是否为空
     * 调用typeof判断值是否为 'object' (原始数据类型)
     * 调用isArray 判断是否为数组
     * 调用 Object.getPrototypeOf 判断值的原型 是否为空 和 是否等于 object的原型(判断是否为原始对象,数组上面判断过了)
     */

    if (!isProxyable(baseState)) // 如果不是原始对象那么就抛出错误
        throw new Error(
            `the first argument to an immer producer should be a primitive, plain object or array, got ${typeof baseState}: "${baseState}"`
        )
        /**
         * `getUseProxies 函数判断当前环境是否存在 Proxy 如果存在就使用 Proxy 如果不存在就使用es5方案
         */
    return getUseProxies()
        ? produceProxy(baseState, producer)
        : produceEs5(baseState, producer)
}


```

阅读文档说明得知 Immer 支持比较旧的 JavaScript 环境, 在比较新的环境下使用性能好的 Proxy ,在比较旧的环境下使用另一种方案先看看 Proxy 方案

# Proxy 方案实现

代码如下

```JavaScript

export function produceProxy(baseState, producer) {

  /**
   *  isProxy 判断 baseState 是否为空如果不为空
   *  判断 baseState中是否存在  __$immer_state(symbol不存在) 属性
   *  如果本身已经被代理过了那么直接运行
   */
    if (isProxy(baseState)) {
        // See #100, don't nest producers
        // 详见 #100  不要嵌套 producers
        const returnValue = producer.call(baseState, baseState) // 调用回调函数,并且将this指向 baseState 本身
        return returnValue === undefined ? baseState : returnValue
    }
    const previousProxies = proxies // proxies = null
    proxies = []
    try {
        // create proxy for root
        // 为 root 创造一个 Proxy
        // createProxy 接收两个参数, 创建一个 proxy代理对象
        const rootProxy = createProxy(undefined, baseState)
        // execute the thunk
        // 执行这个被包装过的对象
        const returnValue = producer.call(rootProxy, rootProxy)
        // and finalize the modified proxy
        // 并最终确认修改后的 proxy
        let result
        // check whether the draft was modified and/or a value was returned
        // 用于检查 draft 是否已经修改或者返回值
        if (returnValue !== undefined && returnValue !== rootProxy) {
            // something was returned, and it wasn't the proxy itself
            // 确定返回了一些东西,而不是 proxy 本身
            if (rootProxy[PROXY_STATE].modified)
            // 如果没有修改或者没有返回抛出错误
                throw new Error(RETURNED_AND_MODIFIED_ERROR)

            // See #117
            // Should we just throw when returning a proxy which is not the root, but a subset of the original state?
            // Looks like a wrongly modeled reducer
            result = finalize(returnValue)
        } else {
            result = finalize(rootProxy)
        }
        // revoke all proxies
        // 完成之后撤销 Proxy
        each(proxies, (_, p) => p.revoke())
        return result
    } finally {
        proxies = previousProxies
    }
}

```

简单的看了一下,发现还是很难,唉还是继续看书吧.
