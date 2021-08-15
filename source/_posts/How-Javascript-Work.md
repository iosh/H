---
title: How-Javascript-Work
date: 2021-08-15 15:39:03
tags: JavaScript
---

how JavaScript work book notes
<!-- more -->

## 命名

命名是自古难题. 业界对于命名单词连接分`驼峰`和`下划线`, 绝大多数语言都不允许在命名中使用空格, JS 也是如此.

## 数值

JavaScript 使用 IEEE 745 标准,作为数字的实现, 很多程序语言都使用这个实现. 由于计算机对小数的二进制存储问题, 无法精准的表示小数.

## 高精度数值

JavaScript 没有 64 位整数类型, 新增的 Bigint 是 54 位有符号整数类型. 书上实关于高精度数值的集中类型都编写了一个范例.

## 布尔类型

由于 JavaScript 有隐形的类型转换, 所以表示 true 和 false 的可以有很多种, 书上对这个行为表示了批判

## 数组

JavaScript 中的数组就是一个对象,并不是其他语言中的数组, 数组有 pop 和 shift 所以在一定程度上也可以模拟队列和栈, JavaScript 的数组提供了很多遍历的方法, 很多实用的方法.

## 对象

JavaScript 中除了 null 和 undefined 之外其他都是对象, 对象也指数据类型. 对象的 key 都是字符类型的, value 可以是 JavaScript 中的任意类型, JavaScript 新增的 Map 类型是真正的映射表, key 可以是人和类型, 比如用一个对象当做 key 或者另一个 Map 当做key.


## 字符串

字符串在本质上是 16 (0-65535) 位无符号整数的不可变数组. 切记在 JavaScript 中字符串是不可变的, 所以 JavaScript 天生支持 Unicode 编码方式,

## 语句

JavaScript 声明可以使用 var(过时) let  const function

## Generator

ES6 引入一个新的特性, generator, Python 里面也有, 而且在 py 代码中还比较常用, 因为可以被挂起和恢复, 所以可以用来实现非常复杂的事情. generator 是有状态的.

## 异常

在 JavaScript 中可以使用 throw 来抛出任意类型的值. 并不一定是 Error 


## 程序

在浏览器中全局变量对象为 window, node 中是 global

## this

老生常谈的 this, 箭头函数没有自身的 this

## 尾调用

函数的结尾返回另外一个函数, 支持度不同 http://kangax.github.io/compat-table/es6/ 支持度列表 proper tail calls (tail call optimisation) 项. 


## 事件化编程

讲了 JavaScript 基于事件的模型, 并提供了一个基于事件化的例子

## JSON

以前都是 XML 现在也有很多服务在使用 XML 传输数据,  因为 JSON 在传输中注释没有用, 并且比较难处理, 所以 JSON 就不支持注释了. 所以用 JSON 作为配置是一个不太好的选择, 没有注释.





