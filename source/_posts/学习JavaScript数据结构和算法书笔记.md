---
title: 学习JavaScript数据结构和算法书笔记
date: 2018-03-31 17:04:01
tags: 数据结构
---

买了一本学习JavaScript数据结构与算法书，记录一下笔记。

<!-- more -->

> 为什么买一本JavaScript的书呢，其实因为C语言的有点难，想从这本书里面学到点东西，然后学习c语言版本的数据结构不是那么难，而且想知道在JavaScript中如何运用到数据结构和算法里面的知识，毕竟在很久一段时间JavaScript都是自己的吃饭工具。数据结构和算法的目的是为了搞笑解决常见问题，并且对日后的代码质量起比较大的作用

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
class Queue {
    constructor(){
        // 声明一个数组保存队列里的元素
        this.items = [];
        }
        // 添加元素到队列末尾
    enqueue (element) {
        this.items.push(element)
    }
        // 移除并返回队列第一个元素
    dequeue() {
        return this.items.shift()
    }
        // 返回队列第一个元素
    front() {
        return this.items[0];
    }
        //  判断元素是否为空
    isEmpty() {
        return this.items.length === 0;
    }
        //  清空队列
    clear() {
        this.items = []
    }
        //  返回队列元素长度
    size() {
        return items.length;
    }
    // 打印队列
    print() {
        console.log(this.items)
    }
}
```

## 使用队列

上面创建了个队列的类，现在生成一个对象，就可以使用它了

```JavaScript
const queue = new Queue()

console.log(queue.isEmpty()) // 返回true，因为队列为空

//现在添加几个元素

queue.enqueue('john')
queue.enqueue('jack')
queue.enqueue('camila')

queue.print() // ["john", "jack", "camila"]

console.log(queue.size()) // 输出3
console.log(queue.isEmpty()) //false 队列不为空

queue.dequeue()
queue.dequeue()
queue.print() // ["camila"] 前面的两个元素已经被删除掉了
// 向队列中添加三个元素


```

## 队列优先

列队大量应用在计算机科学和生活中，这里可以修改上面的列队，让他称为一个优先列队，元素的添加和移除是基于优先级的，现实中的例子，比如医院中的重症患者和普通患者，优先级别是不同的。

要实现一个优先列队，有两种选项：设置优先级，然后在正确的位置添加元素，或者用入列操作添加元素，然后按照优先级移除他们。

```JavaScript
function PriorityQueue() {
   let items = [];
   this.enqueue = function (element) {
        this.items.push(element)
    }
    this.dequeue = function () {
        return this.items.shift()
    }
    this.front = function () {
        return this.items[0];
    }
    this.isEmpty = function () {
        return this.items.length === 0;
    }
    this.clear = function () {
        this.items = []
    }
    this.size = function () {
        return this.items.length;
    }
    this.print = function () {
        console.log(this.items)
    }
    function QueueElenent (element, priority) {
        this.element = element;
        this.priority = priority;
    }

 function QueueElement (element, priority){
  this.element = element;
  this.priority = priority;
 }
 this.enqueue = function(element, priority){
  let queueElement = new QueueElement(element, priority);
  let added = false;
  for (let i=0; i<items.length; i++){
    if (queueElement.priority < items[i].priority){
      items.splice(i,0,queueElement);
      added = true;
      break;
    }
  }
  if (!added){
    items.push(queueElement);
  }
 };
 this.print = function(){
  for (let i=0; i<items.length; i++){
    console.log(`${items[i].element} - ${items[i].priority}`);
  }
 };
}

const queue = new PriorityQueue()

queue.enqueue('john',2)
queue.enqueue('jack',1)
queue.enqueue('camila',1)
queue.print()

// 这里就是就是增加了一个QueueElement类，这个元素包含了要添加列队的元素，他可以是任意类型的，还有本身的优先级

// 如果列队为空，可以直接插入，否则就需要比较这个元素和其他列队元素的优先级，当找到一个优先级值更大（值越大优先级越底）就把元素插入在他之前
```

## 循环队列--击鼓传花

另一种队列的实现就是`循环列队`，列队循环的例子就是击鼓传花游戏，在游戏中，若干小孩围城一个圆圈，把花尽快传递给旁边的人，某一时刻传花停止，这个时候花在谁手中，谁就退出圆圈，结束游戏，重复这个过程，直到只剩一个小孩。

```JavaScript
class Queue {
    constructor(){
        this.items = [];
        }
    enqueue (element) {
        this.items.push(element)
    }
    dequeue() {
        return this.items.shift()
    }
    front() {
        return this.items[0];
    }
    isEmpty() {
        return this.items.length === 0;
    }
    clear() {
        this.items = []
    }
    size() {
        return this.items.length;
    }
    print() {
        console.log(this.items)
    }
}

function hotPotato (nameList, num) {
    const queue = new Queue();
    for (let i = 0; i < nameList.length; i++) {
        queue.enqueue(nameList[i]);
    }

    let eliminated = '';

    while (queue.size() > 1) {
        for (let i = 0; i < num; i++) {
            queue.enqueue(queue.dequeue())
        }
        eliminated = queue.dequeue()
        console.log("被淘汰", eliminated)
    }
    return queue.dequeue()
}
const name = new Array('john','jack','camila','ingrif','carl')
console.log(typeof name)
const winner = hotPotato(name, 7)
console.log(winner)
```

# 链表

数组（可以刻称为列表）是一种非常简单的存储数据序列的数据结构。接下来需要学习如何使用链表和动态的数据结构这意味着可以从中任意添加或者移除向，它会按需进行扩容。

要储存多个元素，数组或者列表可能是最常见的数据结构，这种数据结构非常方便，然而这种结构有一个缺点，在大多数语言中数组的大小是固定的，，从数组中的起点或者中间插入和移除项的成本很高，因为这意味着要移动其他的元素，尽管JavaScript中Array类方法提供了方法，但是背后的情况是一样的。这种成本有时候的代价是高昂的，不可以接受的。

链表储存有序的元素集合，但是不同于数组，链表中的元素在内存中并不是连续放置的，每个元素由一个储存元素本身的节点和一个指向下一个元素的引用（c语言中的指针）组成，相对于数组，链表的一个好处在于，添加或者移除元素的时候不需要移动其他的元素，然而，链表需要使用指针，因此实现链表时，需要额外注意，数组的另一个细节是可以`直接访问任何位置的元素`，然而想访问链表中的一个元素，`需要从七点开始迭代链表直到找到位置`。

## 创建一个链表

使用JavaScript实现一个链表

```JavaScript
class Node {
    constructor(element) {
        this.element = element;
        this.next = null;
    }
}
class LinkedList {
    constructor() {
        // 储存列表项的数量length属性
        this.length = 0;
        // 储存第一个节点的引用
        this.head = null;
    }

    // 向链表尾部添加一个新的元素 （实现第一步）
    append (element) {
      const node = new Node(element);
      let current;
      if (this.head === null) {
        // 如果链表的第一个元素为空
        this.head = node;
      } else {
        // 如果链表不为空
        current = this.head; // 储存链表第一个元素
        while (current.next) { // 迭代链表直到找到链表的结尾
          current = current.next;
        }
        current.next = node;
      }
      this.length++; // 更新链表长度
      // 此时链表的最后一个元素的next会指向空因为Node类预先赋值为noll
    }
    // 向链表指定位置插入一个新的项
    insert (position, element) {}
    // 从链表的特定位置移除一项
    removeAt (position) {}
    // 从链表中移除一项
    remove (element) {}
    // 返回元素在链表中的索引
    indexOf (element) {}
    // 如果链表中不包含任何元素返回true否则返回false
    isEmpty () {}
    // 返回链表包含的元素个数
    size () {}
    // 由于链表项使用了Node类，就需要重写集成于JavaScript对象默认的toString方法，让他只输出元素的值
    toString () {}
}
const list = new LinkedList();
list.append(15)
list.append(18)
console.log(list) // 这个时候测试刚刚实现的append方法会在谷歌控制台
// 得到我们的list实例
// 大概长这个样子
/**
list = LinkedList {
    head: {
        element: 15,
        next: {
            element: 18,
            next: null
        }
    }
    length: 2
}

// 大概就是上面这种结构
*/

```

appemd 方法实现了，接下来实现其他方法

```JavaScript
// 实现 removeAt 方法
class Node {
    constructor(element) {
        this.element = element;
        this.next = null;
    }
}
class LinkedList {
    constructor() {
        // 储存列表项的数量length属性
        this.length = 0;
        // 储存第一个节点的引用
        this.head = null;
    }

    // 向链表尾部添加一个新的元素
    append (element) {
      const node = new Node(element);
      let current;
      if (this.head === null) {
        // 如果链表的第一个元素为空
        this.head = node;
      } else {
        // 如果链表不为空
        current = this.head; // 储存链表第一个元素
        while (current.next) { // 迭代链表直到找到链表的结尾
          current = current.next;
        }
        current.next = node;
      }
      this.length++; // 更新链表长度
      // 此时链表的最后一个元素的next会指向空因为Node类预先赋值为noll
    }
    // 向链表指定位置插入一个新的项
    insert (position, element) {}

    // 从链表的特定位置移除一项
    removeAt (position) {
      if(position > -1 && position < this.length){ // 检查是否越界
        // 判断指定位置是大于-1 和小于链表长度
        let current = this.head, previous, index = 0;
        // 如果制定项是第一项
        if (position === 0){
          this.head = current.next; // 直接让头指针指向第二位
        } else {
          while(index ++ < position) { // 迭代链表
            // 储存要被删除的前一个元素
            previous = current;
            // 储存要被删除的后一个匀速
            current = current.next;
          }
          // 链接前后，被删除的元素被丢弃在内存中等待垃圾回收。
          previous.next = current.next;
          this.length --;
        }
      }
    }
    // 从链表中移除一项
    remove (element) {}
    // 返回元素在链表中的索引
    indexOf (element) {}
    // 如果链表中不包含任何元素返回true否则返回false
    isEmpty () {}
    // 返回链表包含的元素个数
    size () {}
    // 由于链表项使用了Node类，就需要重写集成于JavaScript对象默认的toString方法，让他只输出元素的值
    toString () {}
}
const list = new LinkedList();
list.append(10)
list.append(20)
list.append(30)
list.append(40)
console.log(list) // length 4
list.removeAt(3)
console.log(list) // length 3
```

实现insert方法，任意位置插入一个元素

```JavaScript
class Node {
    constructor(element) {
        this.element = element;
        this.next = null;
    }
}
class LinkedList {
    constructor() {
        // 储存列表项的数量length属性
        this.length = 0;
        // 储存第一个节点的引用
        this.head = null;
    }

    // 向链表尾部添加一个新的元素
    append (element) {
      const node = new Node(element);
      let current;
      if (this.head === null) {
        // 如果链表的第一个元素为空
        this.head = node;
      } else {
        // 如果链表不为空
        current = this.head; // 储存链表第一个元素
        while (current.next) { // 迭代链表直到找到链表的结尾
          current = current.next;
        }
        current.next = node;
      }
      this.length++; // 更新链表长度
      // 此时链表的最后一个元素的next会指向空因为Node类预先赋值为noll
    }
    // 向链表指定位置插入一个新的项
    insert (position, element) {
      if (position >= 0 && position <= this.length) { // 越界检查保证位置合理
        const node = new Node(element);
        let current = this.head, previous, index = 0;

        if (position === 0) {
          // 在第一个位置添加
          node.netx = current; // 将原有链表添加在他后面
          head = node; // 并将头指向这个元素
        } else {
          while (index ++ < position) {
            previous = current;
            current = current.next;
          }
          node.next = current;
          previous.next = node;
        }
        this.length ++
        return true
      } else {
        return false;
      }
    }

    // 从链表的特定位置移除一项
    removeAt (position) {
      if(position > -1 && position < this.length){ // 检查是否越界
        // 判断指定位置是大于-1 和小于链表长度
        let current = this.head, previous, index = 0;
        // 如果制定项是第一项
        if (position === 0){
          this.head = current.next; // 直接让头指针指向第二位
        } else {
          while(index ++ < position) { // 迭代链表
            // 储存要被删除的前一个元素
            previous = current;
            // 储存要被删除的后一个匀速
            current = current.next;
          }
          // 链接前后，被删除的元素被丢弃在内存中等待垃圾回收。
          previous.next = current.next;
          this.length --;
        }
      }
    }
    // 从链表中移除一项
    remove (element) {}
    // 返回元素在链表中的索引
    indexOf (element) {}
    // 如果链表中不包含任何元素返回true否则返回false
    isEmpty () {}
    // 返回链表包含的元素个数
    size () {}
    // 由于链表项使用了Node类，就需要重写集成于JavaScript对象默认的toString方法，让他只输出元素的值
    toString () {}
}
const list = new LinkedList();
list.append(10)
list.append(20)
list.append(30)
list.append(40)
console.log(list)
list.removeAt(3)
console.log(list)
list.insert(2, 100)
console.log(list)

```

接下来思路差不多了，一口气写完

```JavaScript

class Node {
    constructor(element) {
        this.element = element;
        this.next = null;
    }
}
class LinkedList {
    constructor() {
        // 储存列表项的数量length属性
        this.length = 0;
        // 储存第一个节点的引用
        this.head = null;
    }

    // 向链表尾部添加一个新的元素
    append (element) {
      const node = new Node(element);
      let current;
      if (this.head === null) {
        // 如果链表的第一个元素为空
        this.head = node;
      } else {
        // 如果链表不为空
        current = this.head; // 储存链表第一个元素
        while (current.next) { // 迭代链表直到找到链表的结尾
          current = current.next;
        }
        current.next = node;
      }
      this.length++; // 更新链表长度
      // 此时链表的最后一个元素的next会指向空因为Node类预先赋值为noll
    }
    // 向链表指定位置插入一个新的项
    insert (position, element) {
      if (position >= 0 && position <= this.length) { // 越界检查保证位置合理
        const node = new Node(element);
        let current = this.head, previous, index = 0;

        if (position === 0) {
          // 在第一个位置添加
          node.netx = current; // 将原有链表添加在他后面
          head = node; // 并将头指向这个元素
        } else {
          while (index ++ < position) {
            previous = current;
            current = current.next;
          }
          node.next = current;
          previous.next = node;
        }
        this.length ++
        return true
      } else {
        return false;
      }
    }

    // 从链表的特定位置移除一项
    removeAt (position) {
      if(position > -1 && position < this.length){ // 检查是否越界
        // 判断指定位置是大于-1 和小于链表长度
        let current = this.head, previous, index = 0;
        return this.head.element; // 修复删除0返回值不正确
        // 如果制定项是第一项
        if (position === 0){
          this.head = current.next; // 直接让头指针指向第二位
        } else {
          while(index ++ < position) { // 迭代链表
            // 储存要被删除的前一个元素
            previous = current;
            // 储存要被删除的后一个匀速
            current = current.next;
          }
          // 链表前后，被删除的元素被丢弃在内存中等待垃圾回收。
          previous.next = current.next;
          this.length --;
          return current.element
        }
      } else {
        return null
      }
    }
    // 从链表中移除一项
    remove (element) {
      const index = this.indexOf(element);
      return this.removeAt(index);
    }
    // 返回元素在链表中的索引
    indexOf (element) {
      // 如果找到元素就返回index 如果找不到就返回-1
      let current = this.head,index = 0;
      while(current) {
        if (element === current.element) {
          return index;
        }
        index ++;
        current = current.next;
      }
      return -1;
    }
    // 如果链表中不包含任何元素返回true否则返回false
    isEmpty () {
      return this.length === 0
    }
    // 返回链表包含的元素个数
    size () {
      return this.length
    }
    // 由于链表项使用了Node类，就需要重写集成于JavaScript对象默认的toString方法，让他只输出元素的值
    toString () {
      let current = this.head, string = '';
      while (current) {
        string += ` ${current.element}`; // 带点空格好看
        current = current.next;
      }
      return string;
    }
    // 打印链表元素
    getHead () {
      return this.head;
    }
}

```

## 双向链表

链表有很多不同的类型，上面的叫做`单项链表`对应的就有双向链表，在双向链表中，链接是双向的，一个链向下一个链向上

```JavaScript
class Node {
    constructor(element){
        this.element = element;
        this.next = null;
        this.preve = null;
    }
}

class DoublyLinkedList {
    constructor() {
        this.length = 0;
        this.head = null;
        this.tail = null;
    }
}
// 结构改成这个样子
```

那么大体和上面的例子差不多就不单独举例了

```JavaScript

class Node {
    constructor(element){
        this.element = element;
        this.next = null;
        this.prev = null;
    }
}

class DoublyLinkedList {
    constructor() {
        this.length = 0;
        this.head = null;
        this.tail = null;
    }
  insert(position, element){

    //检查是否越界
     if(position >= 0 && position <= this.length){
            var node = new Node(element),
                    current = this.head,
                    previous,
                    index = 0;
            if(position === 0){ // 链表为空直接赋值
                if(!this.head){
                    this.head = node;
                    this.tail = node;
                }else{
                    node.next = current;
                    current.prev = node;
                    this.head = node;
                }
            }else if(position === this.length){ // 链表末尾
                current = this.tail;
                current.next = node;
                node.prev = current;
                this.tail = node;
            }else{
                while(index++ < position){ // 查找链表位置插入
                    previous = current;
                    current = current.next;
                }
                node.next = current;
                previous.next = node;

                current.prev = node;
                node.prev = previous;
            }
            this.length++;
            return true;
        }else{
            return false;
     }
  }
  removeAt (position) {
    if(position > -1 && position < this.length) {
      let current = this.head, previous, index = 0;

      if (position === 0) {
        // 如果是第一项
        this.head = current.next;
        if (this.length === 1) {
          this.tail = null;
        } else {
          this.prev = null;
        }
      } else if (position === this.length -1) {
        current = this.tail;
        this.tail = current.prev;
        this.tail.next = null;
      } else {
        while (index ++ < position) {
          previous = current;
          current - current.next;
        }
        previous.next = current.next;
        current.next.prev = previous;
      }
      this.length --;
      return current.element;
    } else {
      return null
    }
  }
}
const list = new DoublyLinkedList()
list.insert(0, 100)

console.log(list)

```

# 集合

> 集合是一种由无需且唯一（即不能重复）的项组成的，这个数据结构使用了有限集合相同的数学概念。

JavaScript在2015年发布了ECMAscript2015即ES6，其中就包含了set类的实现。

```JavaScript
class Set {
    constructor(){
        this.items = {};
    }
}

/*
这里items使用了对象而不是数组表示集合，但是可以以使用数组
在JavaScript中的对象不允许一个键指向两个不同的属性，这也就保证了
集合里面的元素都是唯一的
*/
```

接下来在class中添加方法

```JavaScript
class Set {
    constructor(){
        this.items = {};
    }

    // 返回布尔值
    has(value) {
        return this.items.hasOwnProperty(value);
    }
    add(value) {
      if(!this.has(value)) {
        this.items[value] = value;
        return true;
      }
      return false;
    }
    remove(value) {
      if(this.has(value)) {
        delete this.items[value]
        return true;
      }
      return false;
    }
    clear() {
      this.items = {}
    }
    size() {
      return Object.keys(this.items).length
    }
    values() {
      return Object.keys(this.items)
    }
}
// 这段代码没什么难度，注释都不需要，思想挺好的，以前都没想到过

```

## 集合操作

对于集合可以进行如下操作

1. 并集：对于给定的两个集合，返回一个包含两个集合中所有元素的新集合

2. 交集：对于给定的两个集合， 返回一个包含两个集合中共有的新集合

3. 差集：对于给定的两个集合， 返回一个包含所有存在第一个集合并且不存在第二个集合的新集合

4. 子集：验证一个给定集合是否是另一个集合的子集

并集：并集的数学概念是集合A和B的并集

```JavaScript
// 实现set 类的并集方法

class SetUnion extends Set {
    union (otherSet) {
        const unionSet = new Set();
        let values = this.values()
        for (let i = 0; i < values.length; i++) {
            unionSet.add(values[i])
        }
        values = otherSet.value()
        for (let i = 0; i < values.length; i++) {
            unionSet.add(values[i])
        }
        return unionSet
    }
}
const setA = new SetUnion()
setA.add(1)
setA.add(2)
setA.add(3)

const setB = new SetUnion()

setB.add(3)
setB.add(4)
setB.add(5)

const unionAB = setA.union(setB)
console.log(unionAB.values())

```

交集： 交集的数学概念是集合A和集合B的交集

实现以下Set类的intersection方法

```JavaScript
class Intersection extends Set {
    intersection (otherSet) {
        const intersectionSet = new Set();
        let values = this.values();
        for (let i = 0; i < values.length; i++) {
            if (otherSet.has(values[i])){
                instersectionSet.add(values[i])
            }
        }
        return instersectionSet;
    }
}
class Intersection extends Set {
    intersection (otherSet) {
        const intersectionSet = new Set();
        let values = this.values();
        for (let i = 0; i < values.length; i++) {
            if (otherSet.has(values[i])){
                intersectionSet.add(values[i])
            }
        }
        return intersectionSet;
    }
}
const setA = new Intersection()
setA.add(1)
setA.add(2)
setA.add(3)

const setB = new Intersection()

setB.add(3)
setB.add(4)
setB.add(5)

const unionAB = setA.intersection(setB)
console.log(unionAB.values())// ['3']
```

差集：差集的数学概念，集合A和集合B的差集，表示A—B

```JavaScript
class Difference extends Set {
    difference (otherSet) {
        const differenceSet = new Set()
        let values = this.values()
        for (let i = 0; i < values.length; i++) {
            if(!otherSet.has(values[i])){
                differenceSet.add(values[i])
            }
        }
        return differenceSet
    }
}
const setA = new Difference()
setA.add(1)
setA.add(2)
setA.add(3)

const setB = new Difference()

setB.add(3)
setB.add(4)
setB.add(5)

const unionAB = setA.difference(setB)
console.log(unionAB.values()) // ['1','2']

```

子集：子集的数学概念，集合A是集合B的子集就是说B包含了A的所有元素

```JavaScript
class Subset extends Set {
    subset (otherSet) {
        if (this.size() > otherSet.size()) {
            return false;
        } else {
            const values = this.values();
            for (let i = 0; i < values.length; i++) {
                if (!otherSet.has(values[i])){
                    return false
                }
            }
            return true;
        }
    }
}
const setA = new Subset()
setA.add(1)
setA.add(2)
setA.add(3)

const setB = new Subset()

setB.add(3)
setB.add(4)
setB.add(5)
const setC = new Subset()
setA.add(1)
setA.add(2)
console.log(setC.subset(setA)) // true
```