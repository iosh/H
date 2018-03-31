---
title: 学习JavaScript数据结构和算法书笔记
date: 2018-03-31 17:04:01
tags: 数据结构
---

买了一本学习JavaScript数据结构与算法书，记录一下笔记。

<!-- more -->

> 为什么买一本JavaScript的书呢，其实因为C语言的有点难，想从这本书里面学到点东西，然后学习c语言版本的数据结构不是那么难，而且想知道在JavaScript中如何运用到数据结构和算法里面的知识，毕竟在很久一段时间JavaScript都是自己的吃饭工具。


> 数据结构和算法的目的是为了搞笑解决常见问题，并且对日后的代码质量起比较大的作用

# JavaScript基础

## 变量

变量保存的数据可以在需要时进行设置，更新或者提取。(基本每一门具备图灵能力的语言都是右变量的)。
在JavaScript中基本数据类型有一下几种：

`Number`数字
`String`字符串
`Boolean`布尔
`Function`函数
`Object`对象
`Symbol`新增表示唯一的

当然还有其他类型，上面是六种JavaScript基本数据类型

## 变量作用域

JavaScript变量作用域就两种一种全局一种局部

## 操作符

这部分也比较简单

常用的：

```JavaScript
算数操作符：+ - * / %(取余) ++ --
赋值操作符：= += -= *= /= %=(余等)
比较操作符：== === != > >= < <=
逻辑操作符：&&(与) || (或) !(非)
位运算：& 按位与 | 按位或 ^按位异或 ～按位非 <<左移 >>右移 // 这一行都是二进制数据操作，有点抽象，不是很好理解，我就记住了一个～
一元操作符：delete 删除一个对象属性或者从数组中删除一个键值
typeof：typeof操作符可以返回一个变量、字符串、关键词或对象的类型
void：void运算符表明一个运算没有返回值，他的返回值是 undefined 我在第三方库中见过用void取得原始undefined值的操作。
in：in 操作符如果所指定的属性确实存在所指定的对象中会返回true
instanceof：如果判断的类型确实是指定的类型则返回true

运算符优先级：太长了优先级最高的是. []这两个，运算级最低的是 ，

```

## 结构控制

条件语句：

    if else

循环

    for
    white
    do white

## 函数

函数是JavaScript的一等公民，函数声明调用传参，还有匿名函数，函数表达式，立即执行函数等等。

## 对象

创建对象两种方式：

    const obj = new Object()
    const obj = {}

## 面向对象编程

在面向对象编程中，对象是一个类的实例（JavaScript没有类，是基于原型的）

声明一个类来表示一本书

```JavaScript
function Book(title, pages, isbn){
    this.title = title;
    this.pages = pages;
    this.isbn = isbn;
}

es6 class
// class不会提升，在严格模式下运行，构造函数可选。
class Book {
    constructor(title, pages, isbn){
        this.title = title;
        this.pages = pages;
        this.isbn = isbn;
    }
}
const book = new Book('H', 200, 2018);
这样就生成一个实例

这里这个new 运算符算一个重点

当new Book()执行时候，会发生下面这些事

1. 一个继承自Book.prototype 的新对象被创建

2. 使用指定的参数调用构造函数Book， 并且将this绑定到新创建的对象，new Book等同于 new Book() 也就是没带参数

3. 由构造函数返回的对象就是new表达式的结果，如果构造函数没有显示返回一个对象，则使用步骤1创建的对象（一般情况下，构造函数不返回值，但是可以主动选择返回对象，用来覆盖正常对象创建的结果）， 这个new值得单独拿出来学习。

```

# 数组

> 数组是最简单的内存数据结构

数组储存一系列同一种数据类型的值，但是JavaScript中数组可以储存任意类型的值，但是这样使用数组并不是良好的习惯。

数组的声明：

    const arr = new Array()
    const arr = new Array(1,2,3,4,5,6,7)
    const arr = []

## 添加删除数组元素

    arr.push(8) // 添加到数组最后
    arr.pop() // 删除最后一个元素
    arr.unshift(0) // 添加到数组最头部
    arr.shift() // 删除数组第一个元素
    arr.splice() // 用来删除现有元素或者添加新元素来更改数组内容

## 二维数组

就数组里面套数组
    [[2][3][4][5]] // 这种

## 数组合并

    arr.concat(arr1)
    [...arr,arr1]

## 数组迭代

    forEach 、 for of 、 map 、every 、some 等方法

## 搜索和排序

sort() 方法，可以根据参数函数对数组进行排序，核心在于排序函数怎么用。V8引擎对sort方法执行两种排序元素小于十个使用插入排序，大于十个使用快速排序。

搜索：indexOf 来查找对应元素的下标。

# 栈

栈是一种遵从先进后出原则的有序集合，新添加或待删除的元素都保存在栈的末尾，称作栈顶，另一端叫做栈底，在栈里，新元素都靠近栈顶，旧元素都接近栈底。

## 栈的创建

创建一个类来表示栈，先声明一个类

    function Stack(){// 各种属性和方法的声明}

首先需要一种数据结构里保存数据结构里的元素，这里使用数组

    cosnt items = []

接下来声明栈的一些方法

push：添加一个或者几个元素到栈顶
pop：移除栈顶部的元素，同时返回移除的元素
peek：返回栈顶部的元素，不对栈做任何修改
isEmpty：如果栈里面没有元素就返回true，否则返回false
clear：移除栈里面的所有元素
size：返回栈里的元素个数

```JavaScript
class Stack {
    constructor(){
        // 声明一个数组保存栈里的元素
        this.items = [];
        }
    // 向栈里添加新的元素
    push (element) {
        this.items.push(element)
    }
    // 移除栈中的元素，遵循先进后出的原则，先出的就是最后元素
    pop () {
        return this.items.pop()
    };
    // 返回栈顶部的元素
    peek () {
        return this.items[this.items.length -1]
    }
    // 判断栈是否为空
    isEmpty () {
        return this.items.length === 0;
    }
    // 返回栈的长度
    size () {
        return this.items.length;
    }
    // 清除栈
    clear () {
        this.items = []
    }
    print() {
        console.log(this.items)
    }
}
// 生成实例
const stack = new Stack()
// 向栈内添加元素
stack.push(5);
stack.push(8);

console.log(stack.peek()) // 8因为8是最后进所以应该最先返回

stack.push(11)
console.log(stack.size()) // 返回栈的长度 3
console.log(stack.isEmpty()) // 查看栈是否为空

stack.push(15)

stack.pop() // 删除11
stack.pop() // 5

console.log(stack.size()) // 2

stack.print() // [5, 8]

下面用构造函数写一下

function Stack() {
    var items = []
    this.push = function (element) {
        items.push(element)
    }
    this.pop = function (){
        return items.pop()
    }
    this.peek = function () {
        return items[items.length - 1]
    }
    this.isEmpty = function () {
        return items.length === 0
    }
    this.size = function () {
        return items.length
    }
    this.clear = function () {
        items = []
    }
    this.print = function () {
        console.log(items)
    }
}

接下来就一样了

```

## 例子十进制到二进制的转换

要把十进制转换成二进制可以将十进制数字除以二（二进制就是满二进一）直到结果为0位置

那么使用上面的栈来实现

```JavaScript
function divideBy2(decNumber) {
    const remStack = new Stack();
    let rem, binaryString = '';

    while (decNumber > 0) {
        rem = Math.floor(decNumber % 2); // 向下取整去到余数
        remStack.push(rem); // 入栈
        decNumber = Math.floor(decNumber / 2); // 修改值再循环
    }

    while (!remStack.isEmpty()) {
        binaryString += remStack.pop().toString() // 栈不为空那么就从栈中依次去出
    }
    return binaryString; // 返回结果
}

这段算法可以修改成十进制转换任意进制

function baseConverter(decNumber, base) {
    const remStack = new Stack();
    let rem, baseString = '', digits = '0123456789ABCDEF'

    while(decNumber > 0) {
        rem = Math.floor(decNumber % base)
        remStack.push(rem);
        decNumber = Math.floor(decNumber / base)
    }

    while(!remStack.isEmpty()){
        baseString += digits[remStack.pop()]
    }
    return baseString;
}

console.log(100,2) // 1100100
console.log(100,10) // 100
console.log(100,8) // 144
console.log(100,16) // 64
```

# 队列

队列是遵循先进先出原则的一组有序列，列队在尾部添加新元素，并从顶部移除元素，最新添加的元素必须在队列为末尾。

在现实中最常见的例子就是排队。

## 创建列队

创建自己的类来表示一个队列和上面的例子非常相似知识添加移除元素原则不同

```JavaScript

```