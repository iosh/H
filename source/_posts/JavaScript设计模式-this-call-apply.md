---
title: JavaScript设计模式-this/call/apply
date: 2017-12-24 15:50:20
tags: JavaScript
categories: 
	- 设计模式
---
**this**

众所周知在JavaScript中的this的指向是基于函数执行的环境**动态绑定**的，而并非是声明时的环境。

**this指向**

在除去不常用的`with`和`eval`具体到实际应用中，`this`的指向可以分为以下四种：
1. 作为对象的方法调用
2. 作为普通函数调用
3. 构造器调用
4. Function.prototype.call和Function.prototype.apply调用
<!-- more -->
**作为对象的方法调用**

作为对象的方法调用this指向该对象

**作为普通函数调用**

当函数不作为对象的属性调用时，也就是常说的普通函数方式，此时的this总是指向全局对象，在浏览器中是window对象(ES6模块会默认启用严格模式，在严格模式下的this指向是未定义并不会指向window)

**构造器调用**

JavaScript中没有类，但是可以从构造器中创建对象，同时也提供了`new`运算符，是的构造器看起来像一个类，通常情况下，构造器里面的this就只想返回的这个对象。（ES6中的class中所有this都指向class本身）

**丢失的this**

这是一个经常遇到的问题， 看下面的代码

```js
const obj = {
	myName: 'sven';
	getName: function(){
		return this.myName;
	}
}
console.log(obj.getName()) // sven
const getName2 = obj.getName;
getName2() // 输出 undefined

```
当调用obj.getName时，getName是作为obj对象的属性被调用的，所以this指向obj，当另一个变量getName2来引用obj.getName，并调用getName2时，此时是作为普通函数调用方式，它的this指向全局window。

**call和apply**

ES3中给Function的原型定义了两个方法，`Function.ptototype.call`和`Function.pototype.apply`，这两个方法的应用也非常广泛。

**call和apply的区别**

`call`和`apply`都是非常常用的方法，他们的作用是一样的，区别仅在于传输参数形式不同。

`apply`接受两个参数第一个参数执行了函数体内`this`对象的指向，第二个参数作为一个带入下标的集合，这个集合可以为数组，也可以为类数组，`apply`方法把这个集合中的元素作为参数传递给被调用的函数。

当调用一个函数时，JavaScript解释器并不会计较形参和实参的数量，类型，以及顺序上的区别，JavaScript的参数实在内部就用一个数组来表示，从这个意义上来说，`apply`比`call`使用率更高，我们不必关心具体有多少参数被传入函数，只要用`apply`一股脑推过去就好了

`call`是包装在`apply`上面的一个语法糖，如果我们明确的知道了函数接受多少个参数，而且想一木了然的表达形参和实参，那么也可以使用`call`来传递参数

这里说一下`bind`，`bind()`方法**创建一个新的函数**，当被调用时，将其this关键字设置为提供的值，在调用新函数时候，在任何提供之前提供一个给定的参数序列。（在react中如果在render之后使用`bind`方法绑定this的话每次刷新都会创建一个新的函数所以请注意）

**call和apply的用途**

**改变this指向**
`call`和`apply`最常见用于改变函数内部`this`指向

```js
const obj1 = {
	name:'sven'
}
const obj2 = {
	name: 'anne'
}
window.name = 'window'
const getName = function(){
	alert(this.name)
}
getName() // window
getName.call(obj1) //sven
getName.call(obj2) //anne
```
现在大部分浏览器都实现了`Function.prototype.bind` 用来指定函数内部的`this`指向下面模拟一下

```js
Function.prototype.bind = function(context){
	var self = this; //保存原来函数
	return function(){
		return self.apply(context, arguments) //执行新的函数时候，会把之前传入的context当做新函数的体内的this
	}
}
```

**借用其他对象的方法**

借用方法的第一种场景是**借用构造函数**，通过这种技术，可以实现一些类似继承的效果

```js
const A = function(name){
	this.name = name
}
const B = function(){
	A.apply(this, arguments);
}
B.prototype.getName = function(){
	return this.name
}
const b = new B('sven')
console.log(b.getName()) // sven
```






