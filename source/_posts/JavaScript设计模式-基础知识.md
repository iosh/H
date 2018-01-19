---
title: JavaScript设计模式-基础知识
date: 2017-12-22 18:23:45
tags: JavaScript
categories: 
	- 设计模式
---

本系列是《JavaScript设计模式与开发实践》一书学习记录
<!-- more -->
###  动态类型语言和鸭子类型
编程语言按照数据类型大体可分为两类 **静态类型语言**（Java）、**动态类型语言**（JavaScript）
静态类型语言在编译的时候便已经确定变量类型，而动态类型语言的变量类型要到运行时候，待变量被赋予某个值之后，才会具有某种类型。(其实微软的typescript可以让JavaScript实现静态类型)。

静态类型语言优点是可以在编译的时候就发现**类型不匹配的错误**，其次，如果在代码中明确规定了数据类型，编译器还可以进行**相关优化**，**提升速度**。缺点首先是强迫程序员依照**强契约**来编写程序，为变量规定**数据类型**，其实只是一种辅助编写可靠性高的一种手段，而不是编写程序的目的。

动态类型语言优点是**编写代码数量更少**，可以把更多精力放在业务逻辑上，缺点是无法保证**变量类型**，比如一个函数接受一个数组参数，但是你传了一个字符串，就会导致函数错误，但是可能只有看过函数代码才知道。（此处说的是用别人的代码，当然大多数库都提供了ts声明文件，帮助编辑器来提示类型，如果没有的话，只能看文档了，如果文档都没写，只能看代码了。）

在js中声明一个变量不需要说明类型，赋值一个字符串他就是一个字符串变量，赋值一个数字他就是数字类型变量，而且可以重复赋值(ES6新增了**const**来声明常量，不可重复赋值）。

这一切都建立在**鸭子类型**的概念上，鸭子类型的通俗说法是：**如果他走路像鸭子，叫起来也像鸭子，那么他就是鸭子**

###  多态

多态的实际含义是：**同一操作用于不同对象上面，可以产生不同的解释和不同的执行结果**，换句话说，给不同的对象发送他容一个消息的时候，这些对象会根据这个消息分别给出不同的反馈。

**一段“多态”的JavaScript代码**
[instanceof运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof),`**instanceof**` **运算符**用来测试一个对象在其原型链中是否存在一个构造函数的 `prototype` 属性。
```js
let makeSound = function(animal){
	if (animal instanceof Dock){
		console.log('嘎嘎嘎');
	}else if (animal instanceof Chicken){
		console.log('咯咯咯');
	};
}
const Dock = function(){};
const Chicken = function(){};
makeSound(new Dock()); // 嘎嘎嘎
makeSound(new Chicken()); // 咯咯咯
```
此段代码确实体现了**多态性**，当分别传入Dock和Chicken代码会返回不同的内容，但是这样的代码是不令人满意的，如果新增一个的话就需要更改makeSound来实现，代码修改的越多，出错的可能性也就越大。

多态背后的思想是将**做什么**和**谁去做以及怎么做**分开，也就是说将**不变的事物**与**可能改变的事物**分离开

**对象的多态性**

首先把不变的部分分离出来，就是所有动物都会叫。
以下代码可以在[这里](https://jsfiddle.net/)进行测试，记得打开控制台才能看到console出来的信息。
```js
let makeSound = function(animal){
	animal.sound();
};
```
然后把可变的部分各自封装起来，刚才的多态性实际上是指对象的多态性：

```js
class Duck{
	sound(){
		console.log('嘎嘎嘎');
	};
}
class Chicken{
	sound(){
		console.log('咯咯咯');
	}
}
makeSound(new Duck()) // 嘎嘎嘎 
makeSound(new Chicken()) // 咯咯咯
```
**类型检查和多态**

类型检查是便显出对象多态性绕不开的话题，JavaScript是一门动态语言类型，书上举了个例子Java的。
```java
String str;
str = 'abc'; // 正常
str = 1;  // 报错与声明不符

把上面的例子换成java的

public class Duck{
	public void makeSound(){
		System.out.println('嘎嘎嘎');
	}
}
public class Chicken{
	public void mackSounde(){
		System.out.println('咯咯咯');
	}
}
public class AnimalSound {
	public void makeSound(Duck duck){
		duck.makeSound()
	}
}
public class Test {
	public static void main(String args[]){
	Duck duck = new Duck();
	animalsound.makeSound(duck)  // 输出 嘎嘎嘎
}
}

```

顺利输出嘎嘎嘎，但是如果让Chicken也打印出来，在java中并不容易实现

```js
public class Test {
	public static void main(String args[]){
	AnimalSound animalSound = new AnimalSound();
	animalsound.makeSound(duck)  // 报错只接受duck类型参数
	}
}
```

为解决这一问题，在静态类型的面向对象语言通常设计为可以向上转型：**当给一个类变量赋值时，这个变量的类型既可以使用这个类本身，也可以使用这个类的超类**，比如说天上有一只喜鹊在飞，如果忽略类型可以说成天上有只鸟。

**使用继承得到多态效果**

使用继承来获得多态效果，是让对象表现出多态性最常用的手段，继承通常报错实现继承和接口继承，这里先说实现继承。(在JavaScript中是没有类的，ES6中的class知识一个语法糖，是基于原型链实现的，class知识让原型的写法更加清晰，更像面向对象的语法而已）

Java代码略

**JavaScript的多态**

多态的思想实际上是把**做什么**和**谁去做**分开，要实现这一点归根节点要消除类型之间的耦合关系。

在JavaScript中变量的类型在运行的时候是可变的，一个JavaScript对象既可以表示Dock类型又可以表示Chicken类型的对象，这意味着**JavaScript对象的多态性是与生俱来的**

这种与生俱来的多态性并不难解释，JavaScript作为一门动态类型语言，他在编译时没有类型检查过程，也没有检查创建的对象类型，又没有检查传递的参数类型。由此尅件某一种动物能否发声，只取决于他有没有mackSounde方法，而不取决于他是某种类型的对象，这里不存在任何程度上的**类型耦合**。

**多态在面上对象程序中的作用**

多态最根本的左右就是通过把过程化的分支语句转化为对象的多态性，从而消除这些条件分支语句。
书上举了个例子

```js
const googleMap = {
	show: function(){
		console.log('开始渲染谷歌地图')
	};
};
const renderMap = function() {
	googleMap.show();
};
renderMap(); // 输出开始渲染谷歌地图
```
出于一些原因要谷歌和百度地图交替使用，修改函数，实现效果

```js
const googleMap = function(){
	show: function(){
		console.log('开始渲染谷歌地图')
	}
}
const baiduMap = function(){
	show: function(){
		console.log('开始渲染百度地图')
	}
}
const renderMap = function(type){
	if(type === 'google'){
		googleMap.show();
	}else{
		baiduMap.show();
	}
}
renderMap('google')
renderMap('baidu')
```
这里就修改完成了，但是很麻烦，如果在添加一个腾讯地图的话先要增加一个qqMap对象然后更新renderMap函数，这样不断地修改导致函数越来越脆弱代码越来越多，也不利于代码维护。

优化一下首先把程序中相同的部分抽象出来，那就是显示地图

```js
const renderMap = function(map){
	if(map.show instanceof Function){// 判断传入对象是否有show方法以及是否是函数
		map.show()
	}
}
renderMpp(googleMap)
renderMap(baiduMap)
```
将显示地图部分抽象出来之后无论新增什么地图，只需要新增一个对象，对象的show方法是调用地图即可。清晰简单。

**封装**

封装的目的是将信息隐藏，一般而言，讨论的封装是封装数据和封装实现。接下来将讨论更广义的耿庄，不仅包括封装数据和封装实现，还包括封装类型和封装变化。

**封装数据**

除了ES6中提供的let之外，一般通过函数来创建作用域
```js
// 这里就不用const let了不然没意义了

var myObject = (function(){
	var _name = 'sven'; // 私有化变量
	return {
		getName: function(){
			return _name;
		}
	}
})()

这个函数是声明一个myObject变量，值是一个自执行函数的结果返回一个对象对象里面有一个getName方法这个方法返回私有化变量_name;其实也是个闭包，myObject变量一直保存着匿名函数的引用，除非主动声明myObject = null; 否则匿名函数会一直在内存中不会被释放。
console.log(myObject.getName()) // sven
```
**封装实现**

上面的封装是数据层面的封装，有时候人喜欢把封装等同于数据封装，但这是一种比较狭义的定义。

封装的目的是将信息隐藏，封装应该被视为**任何形势的封装**，也就是说封装不仅仅是隐藏数据，还包括隐藏实现细节，设计细节以及隐藏对象的类型等。

**封装类型**

封装类型是静态类型语言中一种重要的封装方式，一般而言，封装类型是通过抽象类和结构进行封装的，把对象的真正类型隐藏在抽象类或者接口之后，相比对象类型，用户更关心对象行为，在许多静态语言的设计模式中，想方设法的去隐藏对象的类型，也是促使这些模式诞生的原因之一，比如工厂模式，组合模式。
在JavaScript中，并没有对象类和接口的支持，JavaScript本身也是一门类型模糊的语言（我估计JavaScript之父都没想到JavaScript会发展这么大），在封装类型方面，JavaScript没有能力，也没有必要做的更多，对于JavaScript设计模式实现来说，不区分类型是一种失色，也可以说是一种解脱。

**封装变化**

从设计模式的角度出发，封装在更重要的层面体现为**封装变化**
《设计模式》一书中共归纳总结了23中设计模式，从意图上区分，23种模式分别被划分为创建型模式、结构型模式和行为型模式。
拿创建型模式来说，要创建一个对象，是一种抽象行为，而具体的创建则是可以变化的，创建模式的目的就是封装创建的变化，而结构型模式封装的是对象之间的组合关系，行为模式封装的是对象的行为变化。

通过封装变化的方式，将代码中**稳定不变**部分和**容易变化**的部分隔离开来，在系统的演变过程中，只需要替换那些容易变化的部分，如果这部分是封装好的，替换起来也相对容易。这可以最他程度的保证程序的稳定性和可扩展性。

 **原型模式和原型继承的JavaScript对象系统**

在JavaScript被发明的时候借鉴了Self和Smalltalk两门基于原型的语言，之说以选择基于原型的面向对象系统，是因为从一开始JavaScript中就没有打算加入类的概念。

**使用克隆的原型模式**

从设计模式的角度将，原型模式用于创建对象的一种模式，如果我们想要创建一个对象，一个方法是先指定它的类型，然后通过类来创建这个对象，原型模式选择了另一种方式，找到一个对象，然后通过克隆来创建一个一模一样的对象。

**克隆是创建对象的手段**

原型模式的真正目的并非在于需要得到一个一模一样的对象，而是提供了一种便捷的方式去穿件某个类型的对象，克隆知识创建这个对象的过程和手段。

在JavaScript这种类型模糊的语言中，创建对象非常容易，也不存在类型耦合的问题，从设计模式的角度来讲，原型模式的意义并不大，但JavaScript本事是一门基于原型的面向对象语言，它的对象系统就是使用原型链模式来搭建的，这里称之为原型编程也许更合适。

**体验lo语言**

略

**原型编程规范的一些规则**

原型编程范型至少包括以下基本准则：
1. 所有的数据都是对象
2. 要得到一个对象，并不是通过实例化，而是找打一个对象作为原型并克隆他。
3. 对象会记住他的原型
4. 如果对象无法响应某个请求，他会把这个请求委托给他自己的原型。

**JavaScript中的原型继承**

JavaScript也具有上面的基本规则，就不在重复了。
讨论一下JavaScript是如何在这些规则的基础上来构建它的对象系统。

1. **所有数据都是对象**

JavaScript再设计的时候，模仿了Java引入了两套类型机制： **基本类型**和**对象类型**，其中基本类型包括了`undefined number Boolean string function Object`，从现在来看并不是一个好的想法。

根据设计者的本意，除了`undefined`之外，一切都应是对象，为了我实现这一目标，
`number Boolean string`这几种基本类型也可以通过**包装类**的方式编程对象数据来出处理。

不能说JavaScript中所有数据都是对象，但是可以说绝大部数据都是对象，那么相信JavaScript中也一定有一个根对象。

在JavaScript中根对象是Object.prototype对象Object.prototype对象是一个空白对象，我们在JavaScript中遇到的每个对象，实际上都是从Object.prototype对象克隆而来的，也就是说Object.prototype对象就是他们的原型

[getPrototypeOf](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf) 这个方法返回指定对象的原型内部`[[Prototype]]`属性的值
```js
const obj1 = new Object();
const obj2 = {};
console.log(Object.getPrototypeOf(obj1) === Object.prototype)
console.log(Object.getPrototypeOf(obj2) === Object.prototype)
两个都会返回true
```
2. 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。

在JavaScript语言中并不需要关心克隆细节，这是引擎内部实现的，我们只需要显式的调用`const obj = new Object()`或者`const obj1 = {}`这个时候引擎会从Object.prototype上面克隆一个对象出来，我们最终得到的就是这个对象。

看看如何使用`new` 运算符从构造器中得到一个对象。

```js
class Person{
	constructor(name){
		this.name = name
	};
	getName(){
		return this.name
	}
}
const P = new Person('HXS')
p.name // HXS
P.getName // HXS
```
在JavaScript中并没有类的概念，这句话书中作者重复很多次了（在你不知道的JavaScript书中作者也是强调了很多次），但是刚刚不是`new`了Person吗

在这里Person并不是类，而**函数构造器**，JavaScript中的函数既可以当做普通函数被调用，也可以作为构造函数被调用，当时用`new`运算符来调用函数时，此时的函数就是一个**函数构造器**用`new`运算符来创建对象的过程，实际上也是先克隆`Object.prototype`对象，在进行一些其他的额外操作过程

在谷歌和火狐浏览器等浏览器中对外暴露了`_proto_`对象，可以从以下代码理解以下new运算过程
```js
function Person(name){
	this.name = name;
}
Person.prototype.getName = function(){
	return this.name;
}

var objectFactory = function(){
	var obj = new Object(); // 从Object.prototype上面克隆一个空对象
	Constructor = [].shift.call(arguments) // 取得外部传入的构造器，此例是Person
	obj._proto_ = Constructor.protorype // 指向正确的原型
	var ret = Constructor.apply(obj, arguments)// 借用外部传入的构造器给obj设置属性
	return typeof ret === 'object' ? ret : obj // 确保构造器总是会范湖一个对象
}
var a = objectFactory(Person, 'sven')
a.name // sven
a.getName() // sven

```
3. 对象会记住他的原型

如果请求可以在一个链条中一次向后传递，那么每个节点都必须知道他的下一个节点。
JavaScrip给对象提供了一个名为`_proto_`的隐藏属性，某个对象的`_proto_`属性会默认指向他的构造器原型，在一些浏览器中`_proto_`被公开出来，实际上`_proto_`就是对象根**对象构造器原型**联系起来的纽带。

4. 如果对象无法响应某个请求，它会把这个请求委托给他的构造器的原型

这条规则几十原型继承的精髓所在，当一个对象无法响应某个请求的时候，它会顺着原型链把请求传递下去，直到遇到一个可以处理该请求的对象为止（也有可能找到头也找不到，然后报错）

在JavaScript中每个对象都是从`Object.prototype`对象克隆而来，但对象构造器的原型并不仅限于`Object.prototype`上，而是可以动态指向其他对象，这样一来，当对象a需要借用对象b的能力时，可以有选择性的吧对象a的构造器的原型指向对象b，从而达到继承的效果。

```js
var obj = {
	name: 'sven'
}
var A = function(){}
A.prototype = obj
var a = new A()
a.name // 输出sven
```
执行这个代码引擎做了什么

首先，尝试遍历对象a中的所有属性，但是没有找到name这个属性。
查找name属性这个请求被委托给对象a的构造器原型，他被`a_proto_`记录着并指向了`A.prototypt`而`A.prototypt`被设置为对象obj。
在对象obj中找到了name属性，并返回它的值。

当我们期望得到一个“类”继承另外一个“类”的效果时，往往会使用下面的代码模拟实现：

```js
var A = function(){}
A.prototype = {name: 'sven'};

var B = function(){}
B.prototype = new A()
var b = new B()
console.log(b.name)

```
这段代码执行的时候，引擎又做了什么

首先，尝试遍历对象b中的所有属性，但是没有找到name这个属性
查找name属性的请求被委托给对象b的构造器原型，他被`_proto_`记录着并指向`B.prototype`,而在`B.prototype`被设置成通过`new A()`创造歘来的对象。
但是在该对象中依旧没有找到name这个属性，于是请求继续委托给这个对象构造器的原型`A.prototype`
最终在A.prototype中找到了name属性，并返回了它的值。

**原型继承的未来**

设计模式在很多时候其实都体现了语言的不足之处，Peter Norvig曾经说过，设计模式是对语言不足测补充，如果要是用设计模式，不如去找一门更好的语言。这句话非常正确（书上写的。。）。不过作为web前端开发者，相信JavaScript在未来很长一段时间都是唯一的选择，虽然没有办法更换语言，但是语言本身也在发展，说不定某个模式在下一个版本会成为天然存在，不再需要拐弯抹角的去实现，比如`Object.create`就是原型模式的天然实现，使用`Object.create`来完成原型继承，看起来更能体现出原型模式的精髓。
另外ECMAScript6带来了新的class语法，让JavaScript看起来更像一门基于累的语言，但是其背后仍是通过原型极致来创建对象。


