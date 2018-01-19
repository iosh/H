---
title: JavaScript设计模式-策略模式
date: 2017-12-26 19:23:32
tags: JavaScript
categories: 
	- 设计模式
---

在程序设计中，常常遇到要实现一种功能有多种方案可以选择，比如一个压缩文件的程序，既可以选择zip算法，也可以选择gzip算法。

策略模式的定义：**定义一系列算法，把他们一个个封装起来，并且使他们可以相互转换**。
<!-- more -->
###  使用策略模式计算奖金

策略模式用着广泛的应用，这里书上用了一个计算奖金的例子来举例。

1. 最初的代码实现

编写一个calculateBons 的函数来计算每个人的奖金数额，函数接受两个参数，员工的工资数额和它的绩效考核等级。

```javascript
var calculateBons = function(performanceLevelm, salary){
	if(performanceLevelm === 'S'){
		return salary * 4
	}
	if(performanceLevelm === 'A'){
		return salary * 3
	}
	if(performanceLevelm === 'B'){
		return salary * 2
	}
}
console.log(calculateBons('S', 20000))
console.log(calculateBons('A', 15000))
console.log(calculateBons('B', 10000))
```
这里的代码非常简单，就不写注释了。

这段代码缺陷也十分严重
1. calculateBons函数比较庞大，充斥了if判断语句，这些语句需要覆盖所有逻辑分支
2. calculateBons函数缺乏弹性，如果增加了新的员工，就需要更改函数内部的逻辑，这是违反开放-封闭的原则的。
3. 算法复用性极差，不可复用。


2. 使用组合函数重构代码

最容易想到的就是组合函数来重构代码，把各种算法封装到一个个小函数里，小函数有着良好的命名，可以一目了然知道对应那种算法，也可以被复用。

```javascript
var performanceS = function(salary){return salary * 4}
var performanceA = function(salary){return salary * 3}
var performanceB = function(salary){return salary * 2}

var calculateBonus = function(performanceLevelm, salary){
	if(performanceLevelm === 'S'){
		return performanceS(salary)
	}
	if(performanceLevelm === 'A'){
		return performanceA(salary)
	}
	if(performanceLevelm === 'B'){
		return performanceB(salary)
	}
}
console.log(calculateBonus('A',20000))
```
这里的改善也十分有限，calculateBonus函数依旧可能越来越庞大，而且在系统变化时候缺乏弹性。

3. 使用策略模式重构代码

策略模式指的是定义一系列算法，把他们一个个封装起来，将不变的部分和变化的部分分隔开是每个设计模式的主题，策略模式也不例外。

在这个例子中，算法的使用方式是不变的，而每种绩效规则对应着不同的计算规则是变化的。

使用策略模式来重构上面的代码。第一个版本是模仿传统面向对象语言中的实现，先把每种绩效的计算规则都封装在对应的策略类里面

```javascript
class performanceS{
	calculate(salary){return salary * 4}
}
class performanceA{
	calculate(salary){return salary * 3}
}
class performanceB{
	calculate(salary){return salary * 2}
}

// 定义奖金类

class Bons{
	constructor(html){
		this.salary = null;
		this.strategy = null;
	};
	setSalary(salary){
		this.salary = salary; // 设置员工的原始工资
	}
	setStrategy(strategy){
		this.strategy = strategy; // 设置员工绩效
	}
	getBonus(){
		return  this.strategy.calculate(this.salary)
	}
}
var bonus = new Bons()
bonus.setSalary(10000);
bonus.setStrategy(new performanceB())
console.log(bonus.getBonus()) // 20000
```
重构之后的代码更加清晰，个各类职责鲜明(但是我感觉好麻烦)

###  JavaScript版本中的策略模式

在上面的代码中，让strategy对象从各个策略类中创建而来，这是模拟一些传统面向对象语言的实现，再试及的JavaScript语言中，函数也是对象，所以更简单和直接的做法是吧strategy直接定义为函数

```javascript
var strategies = {
	'S': function(salary){
		return salary * 4	
	},
	'A': function(salary){
		return salary * 3	
	},
	'B': function(salary){
		return salary * 2	
	}
}
var calculateBonus = function(level, salary){
	return strategies[level](salary)
}
console.log(calculateBonus('A', 20000),
calculateBonus('S', 30000)) // 60000 120000
```


###  多态在策略模式中的体现

通过使用策略模式重构代码，消除了源程序中大片的条件分支，所有跟计算奖金有关的逻辑不在放在context中，而是分布在各个策略中，context并没有计算奖金的能力，而是把这个职责委托给某个策略对象，每个策略对象负责的算法被各自封装在对象内部，当我们对这些策略对象发出计算奖金的请求时，他们会返回各自不同的计算结果，这正事对象多态的体现，也是他们可以相互替换的目的，替换context中当前保存的策略对象，便能执行不同的算法来得到我们想要的结果。

###  使用策略模式实现缓动动画

如果让不熟悉前端开发的程序员投票，选出他们眼中的JavaScript语言在web开发中的两大用途，结果可能是这样的
1. 编写一些让div飞来飞去的动画
2. 验证表单

虽然只是玩笑，但是可以看出动画在web开发中的地位。一些别出心裁的动画效果可以让网站增色不少。

有一段时间网页游戏十分流行，（贪玩蓝月。。。）HTML5版本的游戏可以达到不逊于Flash游戏的效果，我们首先让一个小球按照不同的算法进行运动。

###  实现动画效果的原理

用JavaScript实现动画效果的原理跟动画片的制作一样，动画片是吧一些差距不大的原画以较快的帧数播放，达到视觉上的动画效果，在JavaScript中，可以通过连续改变元素的某个css属性。

###  思路和一些准备

目标是编写一个动画类和一些缓动算法，让小球以各种各样的缓动效果在页面中运动。
分析一下思路

1. 动画开始时，小球所在的原始位置
2. 小球的目标位置
3. 动画开始时的准确时间点
4. 小球持续运动的时间

随后通过setInterval创建一个定时器，定时器每隔19ms循环一次，在定时器的每一帧里，我们会把动画已消耗的时间、小球的原始位置、小球目标位置和动画的持续的总时间等信息传入缓动算法，该算法会通过这个几个参数，计算出小球当前应该所在的位置，最后在更新该div对应的css属性，小球就能顺利的运行动起来。

###  让小球运动起来

在实现完整的功能之前，先了解一些常见的缓动算法，这些算法最初来自Flash，但可以非常方便植入其他语言中。

这些算法都接受四个参数，这四个参数的含义分别是动画已消耗的时间、小球原始位置、小球目标位置、动画持续的总时间，返回的值则是动画元素应该处在的当前位置。

```javascript
var tween = {
	linear: function(t, b, c, d){ return c*t/d+b },
	easeIn: function(t, b, c, d){ return c*(t /= d) * t+b },
	strongEaseIn: function(t, b, c, d){ return c*(t /= d) * t *t *t *t +b },
	stringEaseOut: function(t, b, c, d){ return c * ((t = t/d-1)*t*t*t*t+1) +b },
	subeaseIn: function(t, b, c, d){ return c*(t /= d) *t *t +b },
	subeaseOut: function(t, b, c, d){ c*((t = t/d -1)*t *t +1)+b }
}
```
下面编写完整代码，思想来自JQuery库，本节演示策略模式，并非编写一个完整的动画库，所以省去了动画的列队控制等更多完整功能。

```javascript
<body>
	<div style='position:absolute;background:blue'>Im DIV</div>
</body>
```
接下来定义Animate类，Animate的构造函数接受一个参数，即运动起来的dom节点。

```javascript
class Animate{
	constructor(dom){
		this.dom = dom; // 进行运动的dom节点
		this.startTime = 0; // 动画开始时间
		this.startPos = 0; // 动画开始时，don节点的位置，即dom的初始位置
		this.endPos = 0; // 动画结束时，dom节点的位置，即dom的目标位置
		this.propertyName = null; // dom节点需要被改变的css类名
		this.easing = null; // 缓动算法
		this.duration = null; // 动画持续时间
	};
	// start 方法负责启动动画，在动画被启动的瞬间要记录一些信息，供缓存算法在以后计算小球当前位置的时候使用。在记录这些信息之后，此方法还要负责启动的定时器。
	start(propertyName, endPos, duration, easing) {
		this.startTime = +new Date; // 记录启动时间毫秒数
		this.startPos = this.dom.getBoundingClientRect()[propertyName]; // dom节点初始位置
		this.propertyName = propertyName; // dom节点需要被改变的css属性名
		this.endPos = endPos; // dom节点目标位置
		this.duration = duration; // 动画持续时间
		this.easing = twwen[easing]; // 缓动算法
		var self = this;
		var timeId = setInterval(function(){
			if(self.step() === false){ // 启动定时器，开始执行动画
				clearInterval(timeId) // 如果动画已经结束，清除定时器。
			}
		}，19)
	}
}
// start 方法接受一下4个参数
// propertyName 要改变的css属性名，比如left分别表示上下左右移动
// endPos 小球运动的目标位置
// duration 动画持续时间
// easing 缓动算法

//step方法，该方法代表小球运动的每一帧要做的事情，在这里，这个方法负责计算小球当前位置和调用更新css属性的方法update
	step(){
		vat t = +new Date;// 获取当前时间
		if(t >= this.startTime + this.duration){ //1
			this.update(this.endPos) // 更新小球的css属性
			return false
		}
		var pos = this.easing(t - this.startTime, this.startPos,this.endPos - this.startPos, this.duration) // pos为小球当前位置
		this.update(pos) //更新小球css属性
	}
 // 此段代码注释1的意思是，如果当前时间大于动画开始时间加上动画持续是键之和，说明动画已经结束，此时要修正小球的位置，因为在这一帧之后小球的位置已经接近目标位置，但很可能不完全等于目标位置，此时我们要主动修正小球的当前位置为坠重的目标位置，此外让step返回false，通知start方法清除定时器。
	update(pos){
		this.dom.style[this.propertyName] = pos + 'px'
	}
```

###  更广义的“算法”

策略模式指的是定义一系列的算法，并且把他们封装起来。

从定义开，策略模式就是用来封装算法，但如果把策略模式仅仅用来封装算法，未免有一些大材小用，在实际开发中，我们通常会被算法的含义扩散开，是侧罗模式可以用来封装一系列的业务规则。

###  表单验证

表单验证逻辑
1. 用户名不能为空
2. 密码长度不能少于6位
3. 手机号码必须符合格式

```javascript
var stertegies = {
	isNonEmpty: function(value, errorMsg){
		if(value === ''){
			return errorMsg // 判断不为空
		}
	},
	minLength: function(value, length, errorMsg){ // 限制最小长度
		if(value.length < length){
			return errorMsg
		}
	},
	isMobile: function(value, errorMsg){ // 手机号码格式
		if(!/(^1[3|5|8]0-9]{9}$)/.test(value)){
			return errorMsg;
		}
	},
}
// 接下来实现Validator类，作为content负责接收用户的请求，并委托给strategy对象
var Validator = function(){
	this.cache = []
}
Validator.prototype.add = function(dom, rule, errorMsg){
	var ary = rule.split('.') // 把strategy和参数分开
	this.cache.push(function(){ // 把strategy和参数分开
		var strategy = ary.shift() // 用户挑选的strategy
		ary.unshift(dom.value)  // 把input的value添加进参数列表
		ary.push(errorMsg) // 把errorMsg添加进参数列表
		return strategies[strategy].apply(dom, ary)
	
	})
}

Validator.prototype.start = function(){
	for(var i = 0, validatorFunc; validatorFunc = this.cache[i++]){
		var msg = validatorFunc() // 开始校验，并取得校验后返回信息
		if(msg){
			return msg
		}
	}
}
```

###  策略模式的优缺点

策略模式是一种常用有效的设计模式，总结一些优点

1. 策略模式利用组合、委托和多态等技术和思想，可以有效避免多重条件选择语句
2. 侧罗模式提供了对开放-封闭原则的完美支持，将算法独立封装，是的它们易于切换，易于理解，易于扩展。
3. 策略模式中算法可以复用在系统的其他地方，从而避免许多赋值粘贴工作。
4. 侧罗模式中利用组合和委托让Content拥有执行算法的能力，这也是一种继承的一种更轻便的替代方案。

策略模式有一些缺点，但这些并不严重

首先使用策略模式在程序中正价许多策略类或者策略对象，但实际上这比把他们负责的逻辑堆砌在Content中要好。


其次使用策略模式，必须了解所有的策略，必须了解各个策略之间的不同，才能选择一个合适的策略。


