---
title: JavaScript设计模式-单例模式
date: 2017-12-25T19:46:49.000Z
tags: JavaScript
categories: 
	- 设计模式
---

单例模式的定义是：**保证一个类仅有一个实例，并提供一个访问它的全局访问点**

单例模式是一种常用的模式，有一些对象我们往往只需要一个，比如线程池，全局缓存，浏览器中的window对象，在JavaScript开发中，单例模式的用途同样十分广泛，比如，当我们单击登录按钮的时候，页面会出现一个登录浮窗，而这个登录浮窗是唯一的，无论单击多少次登录按钮，这个浮窗都只会被创建一次，那么这个登录浮窗就适合用单例模式来创建。
<!-- more -->
###  实现单例模式
实现一个单例模式，用一个变量来标识当前是否已经为某个类创建过对象，如果是，则在下一次获取该类实例的时候直接返回之前创建的。

```javascript
class Singleton {
 	constructor(name){
		this.name = name; // 初始化变量
		this.instance = null;
	};
	getName(){ // 获取name值的方法
		alert(this.name)
	}
}
Singleton.getInstance = function(name){ // 设置name
		if(!this.instance){ // 判断instance是否为空如果是空就新建一个实例并储存后返回
			this.instance = new Singleton(name)
		}
		return this.instance; // 返回实例
	}
const a = Singleton.getInstance('sven') // 生成唯一实例
const b = Singleton.getInstance('sven2') // 上面已经生成实例了这个不会起作用的
alert(a === b) // true
```
###  透明的单例模式

实现一个“透明”的单例模式，用户从这个类中创建对象的时候，可以像普通的类一样，下面这个例子是负责在页面中创建唯一的div节点

```javascript
var CreateDiv = (function(){
	var instance;
	var CreateDiv = function(html){
		if(instance){
			return instance;
		}
		this.thml = html;
		this.init()
		return instance = this;
	}
	CreateDiv.prototype.init = function(){
		var div = document.createElement('div');
		div.innerHTML = this.html;
		document.body.appendChild(div)
	}
	return CreateDiv
})()
var a = new CreateDiv('seen1') 
var b = new CreateDiv('seen2')
console.log(a === b) // true

```

上面就完成了一个透明的单例类的编写，但是他有一些缺点，为了把instance封装，使用了自执行匿名函数和闭包，并且让这个匿名函数返回真正的Singleton构造方法，增加了程序的复杂度。
观察Singleton构造函数

```javascript
var CreateDiv = function(html){
	if(instance){
		return instance
	}
	this.html = html
	this.init()
	return instance = this
}

```

在这段代码中，CreateDiv的构造函数实际上负责了两件事，第一是创建对象和初始化init方法，第二是保证只有一个对象，但是可以明确的是，这是一种不好的做法，至少这个构造函数看起来很奇怪。
假设某天要用这个类，在页面中创建千千万万个div，既要让这个单例类变成一个普通的可生产许多实例的类，name必须改写CreateDiv构造函数。把控制唯一对象的那段去掉，这种修改会带来不必要的烦恼。


###  用代理实现单例模式

现在引入代理类来解决上面的问题。

还是使用上面的代码，首先在CreateDiv构造函数中，把负责管理单例的代码移除出去，使他成为一个普通的创建div类

```javascript
class CreateDiv{
	constructor(html){
		this.html = html
		this.init()
	};
	init(){
		const div = document.createElement('div')
		div.innerHTMl = this.html
		document.body.appendChild(div)
	}
}
// 接下来引入代理类proxySingletonCreateDiv
var ProxySingletonCreateDiv = (function(){
	var instance;
  	return function(html){
    	if (!instance){
        	instance = new CreateDiv(html)
        }
      return instance
    }
})()

var a = new ProxySingletonCreateDiv('sven1')
var b = new ProxySingletonCreateDiv('sven2')
console.log(a === b)
// 完美一个字符也没有错
```
通过引入代理类的方式，完成了一个单例模式的编写，和之前不同的是，现在我们把负责管理单例的逻辑移到了代理类ProxySingletonCreateDiv中，这样一来CreateDiv就是一个普通类，和ProxySingletonCreateDiv组合起来达到单例模式，这个例子也是缓存代理的应用之一，后面会讲好处。

###  JavaScript中的单例模式

前面提到的几种单例模式的实现，更多的是接近传统面向对象中的实现，单例对象冲“类”中创建而来，在以类为中心的语言中。这是很自然的做法，比如java，如果需要某个对象，就必须先定义一个类，对象重视从类中创建而来。

但是JavaScript是一门无类（class-free）语言，也正因为如此，生搬单例模式的概念并无意义，在JavaScript中创建对象的方法非常简单，既然我们需要一个“唯一”的对象，为什么要先创建类，无异于穿棉衣洗澡，传统的单例模式在JavaScript中并不适用。

>  单例模式的核心是确保只有一个实例，并提供全局访问。

全局变量不是单例模式，但在JavaScript开发中，经常会把全局变量当成单例来使用，比如
```javascript
var a = {}
```
当用这种方法创建对象a时，对象a确实是独一无二的，如果a变量被声明在全局作用域下，则我们可以在代码的任何位置使用这个变量，全局变量提供给全局访问时理所当然的，这样就满足了单例模式的两个条件。

但是全局变量存在很多问题，它容易造成命名空间污染，在大项目中，如果不限制和管理，程序中就会纯在很多这样的变量，JavaScript中的变量很容易被不小心被覆盖。

JavaScript发明者也承认全局变量是一个设计上的失误，在没有足够的时间思考一些东西的情况下导致的后果。

####  1.使用命名空间

适当的使用命名空间，并不会杜绝全局变量，但是可以减少全局变量的数量

最简单的方法依然是用对象字面量方式：

```javasript
var namespce1 = {
	a: function(){
		alert(1)
	},
	b: function(){
		alert(2)
	}
}
```
把a和b都定义为namespace1的属性，这样可以减少变量和全局作用域打交道的机会，另外还可以动态创建命名空间。

```javascript
var MyApp = {}
MyApp.namespace = function(name){
	var parts = name.split('.')
	var current = MyApp
	for(var i in parts){
		if(!current[parts[i]]){
			current[parts[i]] = {}
		}
		current = current[parts[i]]
	}
}
MyApp.namespace('event')
MyApp.namespace('dom.style')

// 上面代码等价于

var MyApp = {
	event： {},
	dom:{
		style: {}
	}
}
```

#### 2. 使用闭包封装私有变量

这种方法把一些变量封装在闭包内部，只暴露一些接口跟外部通信：

```javascript
var user = (function(){
	var _name = 'sven',
		_age = 29
	return {
		getUserInfo: function(){
			return _name + '_' + _age
		}
	}
})()
console.log(user.getUserInfo()) // sven_29
```

###  惰性单例

惰性单例是指在需要的时候才创建对象实例，**惰性单例是单例的重点**，这种技术在实际开发中非常有用，有用的程度可能超出想象。事实上在上面就用过这种技术，instance实例对象总是在调用Singleton.getInstance时候才被创建，而不是页面加载好的时候创建

```javascript
Singleton.getInstance = (function(){
	var instance = null
	return function(name){
		if(!instance){
			instance = new Singleton(name)
		}
		return instance	
	}
})()

```
设置一个登陆窗口点击按钮的时候才创建

```javascript
<html>
	<body>
		<button id='loginBtn'>登陆</button>
	</body>
<script>
	var createLoginLayer = function(){
		var div = document.createElement('div')
		div.innerHTML = '我是登陆弹框'
		div.style.dispaly = 'none'
		document.body.appendChild(div)
		return div
	}
	document.getElementById('loginBtn').onclick = function(){
		var loginLayer = createLoginLayer()
		loginLayer.style.display = 'block'
	}
</script>
</html>

```

上面代码达到了惰性的目的，但是失去了单例的效果，每次点击登陆按钮的时候都会创建一个新的div。

```javascript
var createLoginLayer = (function(){
	var div
	return function(){
		if(!div){
			div = document.createElement('div')
			div.innerHTML = '我是登陆弹窗口'
			div.style.display = 'none'
			document.body.appendChild(div)
		}
		return div
	}
})()
	document.getElementById('loginBtn').onclick = function(){
		var loginLayer = createLoginLayer()
		loginLayer.style.display = 'block'
	}
```
这样改就好了。


###  通用的惰性单例

上面完成了一个可用的的惰性单例，但是有一下问题。
1. 上面的代码仍然是违反单一职责原则的，创建对象和管理单例的逻辑都放在createLoginLayer对象内部
2.  如果我们下次还要创业页面唯一的ifram，或者script标签，用来跨域请求，就得吧createLoginLayer代码几乎重抄一遍。

所以我们需要把不变的部分隔离出来。先不考虑div和iframe有多少差异，管理单例的逻辑其实是完全可以抽离出来的。这个逻辑始终是一样的：用一个变量来表示是否创建过对象，如果是，则下次直接返回这个已经创建好的对象。

```javascript
var obj
if(!obj){
	obj = xxx
}
```

现在把如何管理单例的逻辑从原来的代码中抽离出来，这些逻辑被封装在getSingle函数内部，创建对象的方法fn被当做参数动态传入getSingle函数：

```javascript
var getSingle = function(fn){
	var result
	return function(){
		return result || (result = fn.apply(this, arguments))
	}
}
```

接下来将用于创建登录浮窗的方法fn的形势传入getSingle,我们不仅可以传入createLoginLayer还能传入createScript、createIframe、createXhr 等，之后再让getSingle返回一个新的函数，并且用一个变量result来保存fn的计算结果。result变量因为身在比保重，他永远也不会被销毁。在将来的请求中如果result已经被赋值，那么他将返回这个值。

```javascript
var createLoginLayer = function(){
	var div = document.createElement('div')
	div.innerHTML = '我。登录框'
	div.style.display = 'none'
	document.body.appendChild(div)
	return div
}
var createSingLoginLayer = getSingle(createLoginLayer)
document.getElementById('loginBtn').onclick = function(){
	var loginLayer = createSingLoginLayer()
	loginLayer.style.display = 'block'
}
```

这样就可以了。在这个例子中，吧创建实例对象的职责和管理单例的职责分别放在两个方法里，这两个方法可以独立变化而互不影响，当连接在一起就完成了一个实例对象的功能。。