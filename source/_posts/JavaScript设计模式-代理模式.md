---
title: JavaScript设计模式-代理模式
date: 2017-12-27 20:12:34
tags: JavaScript
categories: 
	- 设计模式
---

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

代理模式是一种非常有意义的模式，在生活中可以找到很多代理模式的场景，比如，明星都有经纪人作为代理，如果详情明星办一场商业演出，那么只能联系他的经纪人，经纪人会把细节谈好，再把合同交给明星。

###  第一个例子，小明追MM的故事

小明遇到了一个百分百女生，两天后小明决定送一朵花表白，小明知道女生有一个闺蜜，于是拜托她送花给女生。

用代理模拟一下过程
```javascript
var Flower = function(){}

var xiaoming = {
	senFlower: function(target){
		var flower = new Flower()
		target.receiveFlower(flower)
	}
}
var A = {
	receiveFlower: function(flower){
		console.log('收到花'+flower)
	}
}

xiaoming.senFlower(A)

// 引入代理B通过B给A送花

var Flower = function(){}

var xiaoming = {
	senFlower: function(target){
		var flower = new Flower()
		target.receiveFlower(flower)
	}
}

var B = {
	receiveFlower: function(flower){
		A.receiveFlower(flower)
	}
}
var A = {
	receiveFlower: function(flower){
		console.log('收到花'+flower)
	}
}

xiaoming.senFlower(B)
```
至此完成一个最简单的代理模式编写。

此处的代理模式其实毫无用处，他做的知识把请求简单的交给本体。

###  虚拟代理实现图片预加载

在web开发中，图片预加载是一种常用的技术，比如直接给img标签节点设置src属性，现用一张loading图片占位，然后异步加载图片，等到图片加载好了再把它填充到img节点。

下面实现这个虚拟代理，首先创建一个普通的本体对象，这个对象负责往页面中创建一个img标签，并且提供一个对外的setSrc接口，外接调用这个接口，便可以给该img标签设置src属性。

```javascript
var myImage = (function(){
	var imgNode = docoment.createElement('img')
	document.body.appendChild(imgNode)
	return {
		setSrc: function(src){
			imgNode.src = src
		}
	}
})()
myImage.setSrc('图片地址')
```
现在开始引用代理对象proxyImage，通过这个代理对象，在图片被真正加载好之前，页面将出现一张占位的loading.gif，来提示用户图片正在加载。

```javascript
var proxyImage = (function(){
	var img = new myImage
	img.onload = function(){
		myImage.setSrc(this.src)
	}
	return {
		setSrc: function(src){
			myImage.setSrc('图片地址')
			img.src = src
		}
	}
})()
proxyImage.setSrc('图片地址')
```

###  代理的意义

为了说明代理的意义，引入一个面向对象的设计原则---单一职责原则。
单一职责原则指的是，就一个类而言，应该仅有一个引起它变化的原因，如果一个对象承担了多项职责，就意味着这个对象将变得巨大，引起他变化的原因可能会有多个，面向对象设计鼓励奖行为分不到细粒度的对象中，如果一个对象承担的职责过多，等于把这些职责耦合到了一起，这种耦合会导致脆弱和低内聚的设计，当变化发生时，设计可能遭到意外的破坏。

###  代理和本体接口的一致性

在客户看来，代理对象和本体是一直的，代理接手请求的过程对于用户来说是透明的，用户并不清楚代理和本体的区别，这样做有两个好处

1.  用户可以放心的请求代理，他只关心是否能得到想要的额结果。
2.  在任何使用本体的地方都可以替换成使用代理。

###  虚拟代理合并HTTP请求

在web开发中也许开销最大的就是网络请求，下面通过一个例子合并请求。

现在页面放置checkbox节点，点击一个按钮就会触发一个请求

```javascript
<body>
<input type='checkbox' id='1' />
<input type='checkbox' id='2‘ />
<input type='checkbox' id='3' />
<input type='checkbox' id='4' />
<input type='checkbox' id='5' />
<input type='checkbox' id='6' />
<input type='checkbox' id='7' />
<input type='checkbox' id='8' />
<input type='checkbox' id='9' />
</body>
// 给CheckBox绑定点击事件，并且点击的同事往另一台服务器同步文件

var synchronousFile = function(id){
	console.log('开始同步文件id为'+id)
}
var checkbox = document.getElementsBytagName('input')
for(var i = 0,c; c = checkbox[i++]){
	c.onclick = function(){
		if(this.checked === true){
			synchronousFile(this.id)
		}
	}
}
```
选择一个CheckBox就会触发一次请求，以我单身25年手速，一秒可以点九个，由此可见频发触发网络请求将会带来相当大的开销

解决方案是通过一个代理函数proxySynchronousFile来手机一段时间之内的请求（函数节流？）

```javascript
var synchronousFile = function(id){
	console.log('开始同步'+id)
}
var proxySynchronousFile = (function(){
	var cache = [],//保存一段时间内需要同步的id
		timer; // 定时器
	return function(id){
		cache.push(id)
		if(timer){ // 保证不会覆盖已经启动的定时器
			return
		}
	}
	timer = setTimeout(function(){
	synchronousFile(cache.join(','))
	timer = null
	cache.length = 0	
	}, 1000)
})()
var checkbox = document.getElementsByTagName('input')
for(var i = 0, c; c=checkbox[i]; i++){
	c.onclick = function(){
		if (this.checked === true){
			proxySynchronousFile(this.id)
		}
	}
}
```

###  缓存代理

缓存代理可以为一些开销大的运算结果提供暂时的储存，下次运算时，如果传递进来的参数和以前一直，则返回前面储存的运算结果。

1.  乘积计算
```javascript
var mult = function(){
	console.log('开始计算乘积')
	var a = 1
	for(var i = 0,l = arguments.length; i<l; i++){
		a = a * arguments[i]
	}
	return a
}

mult(2,3) // 6
mult(3,3,5) // 24

// 加入代理函数

var proxyMult = (function(){
	var cache = {}
	return function(){
		var args = Array.prototype.join.call(arguments, ',')
		if (args in cache){
			return cache[args]
		}
		return cache[args] = mult.apply(this, arguments)
	}
})()

proxyMult(1,2,3,4)
proxyMult(1,2,3,4)

```




