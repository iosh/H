---
title: javaScript递归函数
date: 2017-12-08 19:54:00
tags: javaScript
---

今天找了一些资料重温了一下JavaScript的递归函数，写个博客记录一下。

首先什么是递归
<!-- more -->
>  递归是一项非常重要的编程技巧，函数通过它调用其本身。 ---msdn文档介绍

举个例子,常用的乘阶。

```javascript
function factorial(i) {
	if (i === 1) return i;
	return i* factorial(i - 1);
}
console.log(factorial(5)); // 5*4*3*2*1
```
分几个步骤
1. 声明一个函数factorial，接受一个参数i。
2.	判断i是否等于1，如果等于1则直接返回i。
3.	如果i不等于1，则返回i * factorial(i - 1)，再次调用函数本身。(如此如果严格可以判断i是否大于等于0）
然后函数就会重复2 - 3两个步骤，直至i减到1为止。

本身递归调用到此结束，在学习的过程中发现了其他资料。

###  JavaScript执行上下文栈

看的是这个[教程](https://github.com/mqyqingfeng/Blog/issues/4)

JavaScript执行顺序。
```JS
var foo = functiom () {
	console.log('foo1')；
}
foo();

var foo = functiom () {
	console.log('foo2')；
}
foo();
最后会打印出
foo1
foo2
```
解释一下这个为什么不会被覆盖。
```js
上面的代码写成这样就比较好理解
var foo;
foo = function() {
	console.log('foo1')
}
foo()
foo = function() {
	console.log('foo2')
}
foo()

```
```js
function foo() {

    console.log('foo1');

}

foo();  // foo2

function foo() {

    console.log('foo2');

}

foo(); // foo2
```
在解释一下这个
```js
function foo() {

    console.log('foo1');

}
function foo() {

    console.log('foo2');

}
foo();  // foo2
foo();  // foo2
```
函数被提升之后第二个覆盖了第一个，这是JavaScript作用域提升。
JavaScript的可执行代码有三种，全局代码，函数代码，eval代码
当执行到一个函数的时候就会进行准备工作，叫做`执行上下文`。
原博客把执行上下文说的很清楚了，我理解了一部分。有兴趣可以点击上面的连接继续了解，继续讲递归。
###  尾调用

根据上面的知识，以及以前的知识，我们都知道递归会消耗大量内存，之执行一个函数就压入上下文栈，直至递归结束才会释放，造成递归占用大量内存。

原博客介绍
>  尾调用，是指函数内部的最后一个动作是函数调用，改调用的返回值，直接返回给函数。

```js
	function f (x) {
		return g(x)
	}
	// 尾调用
```
```js
	function f(x) {
		return g(x) - 1
	}

	// 非尾调用
```

用上面的上下文栈，来看第二个函数，函数f执行指挥返回一个g函数而g函数的结果需要f函数作用域内 - 1才是结果，导致引用的时候g函数入栈的时候会持有f函数的作用域，f得不到释放，一直等g函数结束之后才会被释放。


现在优化一下上面的递归

```js
function factorial(i, res) {
	if(i === 1) return res;
	return factorial(i-1, i*res)
}
console.log(factorial(4, 1)) 
```
这样优化函数，保持返回的函数没有上个函数的引用，这样上个函数在入栈之后执行到return之后就会被释放，而不会和上面的递归一样等到最终结果才会被释放。（ps。JavaScript上下文栈，需要好好看看比较重要）


