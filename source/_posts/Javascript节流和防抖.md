---
title: Javascript节流和防抖
date: 2017-12-09 15:32:45
tags: JavaScript
---


###  JavaScript函数防抖

在前端开发中可能会遇到一些频发触发的事件，例如
1.	window 的 resize(浏览器窗口或者HTML对象改变大小)、scroll(滚动条事件）
2.	mousedown(鼠标在元素上按下事执行事件)、onmousemove(鼠标指针移动到元素上触发)
3.	keyup(按键被释放触发)、keydown(按键被按下触发)
<!-- more -->
举例

```HTML
HTML
<style>
	#root{
		width: 100%; height: 200px; line-height: 200px; text-align: center; color: #fff; background-color: #444; font-size: 30px;
	}
</style>
<div id='root'></div>
<script src='./demo.js'></script>

```
```js
dome.js

var conut = 1; // 声明一个变量储存触发次数
var root = document.getElementById('root'); // 获取id为root的div
function getUserAction() { // 声明一个函数更改root的HTML为conut的值并将conut值++
		root.innerHTML = conut++;
	}

root.onmousemove = getUserAction;

```
效果如下


![](https://github.com/mqyqingfeng/Blog/raw/master/Images/debounce/debounce.gif)


从图上可以看到从左边到右边一共触发了160多次onmousemove事件，同样执行了160多次getUserAction函数。
因为这个例子简单，浏览器可以处理的过来，如果换成请求， 频繁触发就会出现卡顿。

修改一下

```js
 dome.js

var conut = 1; // 声明一个变量储存触发次数
var root = document.getElementById('root'); // 获取id为root的div
function getUserAction() { // 声明一个函数更改root的HTML为conut的值并将conut值++
		root.innerHTML = conut++;
	}
function debounce(fn, wait) { // 声明一个函数，函数接受两个参数。
	var timeout; // 声明一个变量储存定时器的返回值；
	return function() {
		clearTimeout(timeout) 函数被执行了先清除先前定时器的返回值(传入一个错误的ID 给 clearTime 不会有任何影响（也不会抛出异常）)
        timeout = setTimeout(fn, wait); // 定时器会返回一个id给timeout
	}
}
root.onmousemove = debounce(getUserAction, 1000);

```
上面这个函数

分析一下上面的操作
1. 鼠标移动触发onmousemove事件。
2. onmousemove事件调用debounce函数
3.	debounce函数接收到getUserAction， 1000函数之后返回一个匿名函数-清除前面的定时器-执行新的定时器

然后只要鼠标移动就会1-2-3执行，直至鼠标停留超过一秒，才会执行getUserAction函数。

###		JavaScript节流

节流就是如果持续触发事件，则每个一段时间，只执行一次事件。

主流有两种方法：`使用时间戳`， `设置定时器`

使用时间戳

使用时间戳，当事件触发时，去除当前的时间戳，然后减去之前的时间戳，最开始可以设置为0，如果大于设置的时间周期，就执行函数，然后更新时间戳为当前时间戳，如果小于，则不执行。

```js
function throttle(fu, wait) {
	var previos = 0;
	return () => {
		var now = +new Date(); // new Date()做一个+运算，触发对象执行valueOf进行求值得到为毫秒数
		if (now - previous > wait) { // 判断当前的毫秒数减去预制的毫秒数是否大于传入参数的设置
            fu(); // 执行传入的函数
            previous = now; // 更新储存的毫秒数为此次执行的
		}
	}
}

root.onmousemove = throttle(getUserAction, 1000);
```

使用定时器

当事件被触发的时候设置一个定时器，在触发事件，如果定时器存在则不执行，直至定时器执行然后执行函数，清空定时器。

```js
function throttle(fn, wait) {
	var timeout; // 声明一个变量储存定时器返回值
	return () => {  // 返回一个函数
		if(!timeout) { // 判断timeout是否存有定时器返回值
			timeout = setTimeout(()=>{
				timeout = null;// 定时器执行
				fn(); // 执行传入函数
			}, wait) // 延迟多久
		}
	}
}
root.onmousemove = throttle(getUserAction, 1000);
```
