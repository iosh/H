---
title: JavaScript设计模式-闭包和高阶函数
date: 2017-12-24 17:42:09
tags: JavaScript
categories: 
	- 设计模式
---

虽然JavaScript是一门完整的面向对象的编程语言，但这门语言同时也拥有许多函数式语言的特性。（天天听大佬讲函数式编程）
<!-- more -->
**闭包**

对于JavaScript程序员来说闭包是一个难懂又必须征服的概念，闭包的形成与变量的作用域及变量的生存周期密切相关。

**变量的作用域**

变量的作用域，就是指变量的有效范围，当函数中声明一个变量的时候，如果该变量前面没有带关键字`var`，那么这个变量就会成为全局变量。
另外一种情况是用`var`关键字在函数中声明变量，这个时候的变量即是**局部变量**，只有在该函数内部才能访问到这个变量，在函数外部是访问不到的。

举例代码略，都懂。

**变量的生存周期**

除了变量的作用域之外，另外一个和闭包息息相关的概念是变量的**生存周期**
对于全局变量来说，全局变量的生存周期是**永久**的，除非主动销毁。
而对于在函数内用`var`关键字声明的局部变量来说(`const``let`也是一样的）,当退出函数时，这些局部变量即失去了他们的价值，他们都会随着函数调用的结束而被**销毁**。

这段代码也是基础略

**闭包的更多作用**

**封装变量**

闭包可以帮助吧一些不需要暴露在全局的变量封装成**私有变量**。

**延续局部变量的寿命**

由于外部变量一直会保有闭包的引用，所以闭包里面的变量会一直被保留在内存。

**闭包和面向对象设计**

过程与数据的结合是形容面向对象中的**对象**时经常使用的表达式，对象以方法的形式包含了过程，而闭包则是在过程中以环境的形势包含了**数据**。通常用面向对象思想能实现的功能，用闭包也能实现。

看一段跟闭包相关的代码。
```js
const extent = function(){
	let value = 0;
	return {
		call: function(){
			value++;
			console.log(value)
		}
	}
}
const extents = extent()
extents.call()//1
extents.call()//2
extents.call()//3
extents.call()//4
```
如果换成面向对象的写法就是(学习了以前没这么写过，真的蠢）

```js
const extent = {
	value:0,
	call: function(){
		this.value++;
		console.log(this.value)
	}
}
extent.call() // 1
extent.call() // 2
extent.call() // 3
```
**用闭包实现命令模式**

在JavaScript版本的各种设计模式实现中，闭包的运用非常广泛。

代码略

在JavaScript中，函数作为一等对象，本身就可以四处传递，用函数对象而不是普通对象来封装请求显得更加简单和自然，如果需要往函数对象中预先植入命令的接收者，那么闭包可以完成这个工作。

代码略

**闭包与内存管理**

闭包是一个非常强大的特性，但是有很多误解，一种比较常见的就是**闭包会造成内存泄漏，所以要尽量减少闭包的使用**。

局部变量本来应该**在函数退出的时候被解除引用**，但如果局部变量被封闭在闭包形成的环境中，**那么这个局部变量就能一直生存下去**，从这个意义上看，闭包的确会使一些数据无法被**及时销毁**。使用标的一部分原因是**我们选择主动把一些变量封闭在闭包中**，因为可能以后还需要**使用这些变量**，把这些变量放在闭包中和放在全局作用域，**对内存方面的影响是一致的**，这里并不能说成内存泄漏，如果在将来需要回收这些变量，**我们可以手动把这些变量设置为**`null`。

跟闭包和内存泄漏有关系的地方是，使用闭包的同时**比较容易形成循环引用**，如果闭包的作用域链中保存一些DOM节点，这个时候就可能造成内存泄漏，但**这个问题本身并不是闭包的问题**，也非JavaScript的问题。

同样如果要解决循环引用带来的内存泄漏问题，**只需要把循环引用中的变量设置为`null`,这样就切断了连接，垃圾收集器下次运行时，就会回收（书上说的是ie浏览器的计数垃圾回收机制）

**高阶函数**

高阶函数是指至少满足下列条件之一的函数

1. 函数可以作为参数被传递
2. 函数可以作为返回值输出

JavaScript语言中的函数显然是满足高阶函数的条件，在实际开发中，无论是将函数作为参数传递，还是让函数执行结果返回另外一个函数，这两种清秀都有很多应用场景。

**函数作为参数传递**

把函数当做阐述传递，这代表我们可以抽离一部分容易变化的业务逻辑，把这部分业务逻辑放在一个函数参数中，这样一来可以分离业务代码中变化和不变化的部分。

1. 回调函数

都懂，不抄代码了。

2. Array.prototype.sort

`sort`函数接受一个函数当做参数，这个函数里面封装了数组元素的排序规则。

```js
[1,4,3].sort(function(a,b){
	return a-b
})
输出[1,3,4]
```

**函数作为返回值输出**
```js
const getSingle = function(fn) {
	let ret;
	return function(){
		return ret || (ret = fn.apply(this, arguments))
	}
}
```
这个高阶函数的例子，既可以把函数当做参数传递，又让函数执行后反悔了另一个函数。看看函数怎么用

```js
const getScript = getSingle(function(){
	return document,createElement('script')
})

const script1 = getScript()
const script2 = getScript()
console.log(script1 === script2) // true
```
**高阶函数实现AOP**

AOP（面向切面编程）的作用是把一些跟核心业务逻辑无关的功能抽离出来，这些跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等。
java代码略

**高阶函数的其他应用**

1. currying

首先讨论的就是**函数柯里化**。

**currying**又称**部分求值**，一个**currying**的函数首先会接受一些参数，接受了这些参数之后，该函数并不会立即求值，而是继续返回另一个函数，刚才传入的参数在函数形成的比保重保存起来，待到函数被真正需要求值的时候，之前传入的所有参数会被一次性用于求值。

计算每月开销函数，如果在前29天，我们都只是保存好当天的开销，直到30天才进行求值计算，这样就达到了要求，下面的函数并不是一个完整的currying函数，但是有利于理解其中的思想。

```js
const cost = (function(){
	let args = []; // 储存每天的消费
	return function(){ // 返回一个函数
		if(arguments.length === 0){ // 判断传入参数是否为空如果为空则求和
			let money = 0;
			for(let i = 0, l = args.length; i<l; i++){
				money += args[i];
			}
			return money // 返回金额
		}else{
			[].push.apply(args, arguments) // 如果有参数则是在记录
		}
	}
})()
cost(100） //有参数则函数记录
cost(100)
cost(100)
const() //无参数则返回总和 300
```
接下来编写一个通用的函数柯里化。`function currying(){}`接受一个参数，即将被currying的函数，在这个例子中这个函数的作用遍历本月每天的开销并求出他们的总和。

```js
 // 代码整体逻辑是和上面差不多的
const currting = function(fn){
	let args = [];
	return function curryings(){
		if(arguments.length === 0){
			return fn.apply(this, args)
		}else{
			[].push.apply(args,arguments)
			return curryings // 这里原书上写的是argument.callee这个在严格模式下是被删除了，所以把匿名函数给个名字然后返回这个函数
		}
	}
}
const cost = (function(){
	let money = 0;
	return function(){
		for(let i = 0, l = arguments.length; i<l; i++){
			money += arguments[i]
		}
		return money
	}
})()
const costs = currying(cost)
costs(100) //100
costs(100) //200
costs(100) //300
costs(100) //400
costs() //400
```
2. uncurrying
在JavaScript中，当我们调用对象的某个方法是，其实不用关心该对象原本被设计为拥有这个方法，这也是动态类型的特点，也是常说的鸭子类型思想。

同理，一个对象也未必只能使用它自身的方法，name有什么办法可以让对象去借用一个原本不属于它的方法，答案是使用`call`和`apply`都可以完成这个需求。

```js
const obj1 = {
	name: 'sven'
}
const obj2 = {
	getName: function(){
		return this.name
	}
}
console.log(obj2.getName.call(obj1)) // sven
console.log(obj2.getName.apply(obj1)) // sven
```

常用十足对象借用`Array.prototype`的方法，这是`call`和`apply`最常见的应用场景

```js
(function(){
	Array.prototype.push.call(arguments, 4) // arguments借用Array.prototype.push方法
	console.log(arguments) // 输出一个数组[1,2,3]
})(1,2,3)
```
那么有没有办把泛化this的过程提取出来，`uncurrting`就是用来解决这个问题，一下代码是`uncurrting`的实现方法之一

```js
Function.prototype.uncurrying = function(){
	var self = this;
	return function(){
		var obj = Array.prototype.shift.call(arguments)
		return self.apply(obj, arguments)
	}
}
```
在类数组`arguments`借用`Array.prototype`方法之前先把`Array.prototype.push.call`这句话转换一个通用的push函数

```js
var push = Array.prototype.push.uncurrying()
(function(){
	push(arguments, 4)
	consolt.log(arguments) // 输出[1,2,3,4]
})(1,2,3)
```
通过`uncurrting`的方式，`Array.prototype.push.call`变成了一个通用的`push`函数。这样一来push函数作用就和`Array.prototype.push`一样了，同样不仅仅局限于只能操作Array对象。

**函数节流**

前面有博客文章介绍略

**函数防抖**

前面有博客文章介绍略






