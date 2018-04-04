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

# 字典和散列表

## 字典

集合、字典和散列表可以表示一组互不相同的元素(不重复)的值，在集合中重要的是每个值本身，并把他当做主要元素， 在字典中使用[键，值]的形式储存数据，在散列表中也是一样，但是两种数据的结构实现方式略有不同。

### 创建一个字典

和set类似，ES6也已经实现了Map结构，这就是所说的字典结构。
实现一个比Map简单的。

```JavaScript
class Dictionary {
    constructor() {
        this.items = {}
    }
    has (key) {
        return this.items.hasOwnProperty(key)
    }
    set (key, value) {
        this.items[key] = value
    }
    remove (key) {
        if (this.has(key)){
            delete this.items[key];
            return true
        }
        return false
    }
    get (key) {
        return this.has(key) ? this.items[key] : undefinde
    }
}
const dictionary = new Dictionary()
dictionary.set('gandalf','gandelf@qq.com')
dictionary.set('john','john@qq.com')
dictionary.set('tyrion','tyrion@qq.com')
console.log(dictionary)
// es6的Map数据结构，是真正的值-值，因为普通对象的键只能是字符串，而es6的Map
// 键可以是任何内容，数字啊，字符串啊，甚至dom都可以
```

### 散列表

`散列算法`的作用是尽可能快的在数据结构中找到一个值，上面的类中需要找到一个值，需要遍历整个数据结构来找到他，如果使用`散列函数`就知道值的具体位置，因此能够快速检索到该值，散列函数的作用是给定一个键值，然后返回值在表中的地址。

### 创建一个散列表

```JavaScript
class HashTable {
  constructor() {
    this.table = [];
  }
  // 散列函数
  liseloseHashCode (key) {
    let hash = 0;
    console.log(key)
    for(let i = 0; i < key.length; i++) {
      hash += key.charCodeAt(i);
    }
    return hash % 37; // 为了返回一个比较小的值。
  }
  put (key, value) {
    const position = this.liseloseHashCode(key);
    this.table[position] = value
  }

  get (key) {
    return this.table[this.liseloseHashCode(key)];
  }

  remove (key) {
    this.table[this.liseloseHashCode(key)] = undefinde
  }
}

const hash = new HashTable()
hash.put('gandalf','gandelf@qq.com')
hash.put('john','john@qq.com')
hash.put('tyrion','tyrion@qq.com')
console.log(hash.get('john'))
// 这里数组储存了三个元素，但是占用了25个空间，感觉很不合理。
```

# 树

目前为止学习的都是一些`顺序`数据结构，第一个非顺序数据结构是散列表，接下来学习另一种非顺序数据结构-树，对于储存需要快速查找的数据非常有用。

树是一种分层数据的抽象模型，现实生活中最常见的树的例子就是家谱，系统目录等。

## 树的相关术语

一个树结构包含一系列存在父子的节点，每个节点都有一个父节点，除了顶部的第一个节点以及零个或者多个子节点。

```
                   11
                /     \
              /         \
            /             \
          7                15
       /     \         /      \
      5       9       13      20
     / \     / \     /  \    /  \
    3   6   8   10  12  14  18  25
```

位于顶部的11叫做根节点，树中的每个元素都叫做节点，节点分为内部节点和外部节点，至少右一个子节点的节点称为内部节点7,5,9,15,13,20 都是内部节点，没有子元素的节点称为外部节点或者叶节点3,6,5,10,12,14,18,25都是叶节点。

一个节点可以右祖先和后端，一个节点（除了根节点）的祖先包括父节点，祖父节点，曾祖父节点等，一个节点的后代包括子节点，孙子节点，曾孙节点等，例如8的祖先节点右7和11，后代节点右3和6

另外一个有关的术语是子树，子树由节点和它的后代构成，例如13,12,14构成了一棵子树

节点的一个属性是深度，节点的深度取决于它的祖先节点数量，比如节点3有三个祖先节点，他的深度为3.

树的高度取决于所有节点深度的最大值，一棵树也可以被分解成层级，根节点在0层他的子节点在1层，以此类推上面的树高度为3.

## 二叉树和二叉搜索树

二叉树的每个节点只能有两个节点：一个左侧节点，另一个右侧节点，这些定义有助于写出更高效的对树的操作，二叉树在计算机科学中应用很广。（二叉树不能右重复节点，比如系统目录同目录下不能有重名文件夹）

二叉搜索树BST是二叉树的一种，但是他只允许你在左侧节点储存比父节点小的值，右侧节点储存比父节点大的值。

实现一个二叉树

```JavaScript
class Node {
  constructor(key) {
    this.key = key;
    this.left = null;
    this.right = null;
  }
}

class BinarySearchTree {
  constructor() {
    this.root = null;
  }
  // insert 向树中插入一个节点
  insert (key) {
    const newNode = new Node(key);
    if (this.root === null) {
      // 如果树为空
      this.root = newNode;
    } else {
      this.inserNode(this.root, newNode)
    }
  }
  
  // 查询插入函数
  
  inserNode (node, newNode) {
    // 先判断比当前节点大（右）还是小（左）分别去左或者去右
    if (newNode.key < node.key) {
      // 如果left为空直接赋值
      if (node.left === null) {
        node.left = newNode;
      } else {
        // 不为空递归调用
        this.inserNode(node.left,newNode);
      }
    } else {
       // 思路和上面一样
      if (node.right === null) {
        node.right = newNode;
      } else {
        this.inserNode(node.right, newNode);
      }
    }
  }
  
  
}
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
console.log(tree)
```

## 树的遍历

遍历一棵树是指访问树的每个节点，并对他们进行某种操作的过程，常用有三种树的遍历方法

### 中序遍历

中序遍历是一种以上行顺序访问BST所有节点的遍历方式，也就是从最小到最大的顺序访问所有节点，中序遍历一种应有就是对树进行排序操作

```JavaScript
class InOrderTraverse extends BinarySearchTree {
    inOrderTraverse (node, callback) {
    console.log(node)
        if (node !== null) {
            this.inOrderTraverse(node.left,callback)
            callback(node.key)
            this.inOrderTraverse(node.right,callback)
        }
    }
}
const traverse = new InOrderTraverse()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
traverse.inOrderTraverse(tree.root, (key) => console.log(key))
```

### 先序遍历

先序遍历是以优先于后代节点的顺序访问每个节点，先序遍历一种应用是打印结构文档

```JavaScript 
class PreorderTraverseNode extends BinarySearchTree {
    preOrderTraverse (root,callback) {
      this.preOrderTraverseNode(root,callback)
    }
    preOrderTraverseNode(node, callback) {
      if (node !== null) {
        callback(node.key);
        this.preOrderTraverseNode(node.left,callback);
        this.preOrderTraverseNode(node.right, callback)
      }
    }
}
const a = new PreorderTraverseNode()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
console.log(tree)
a.preOrderTraverse(tree.root,(key) => console.log(key))
```

### 后序遍历

后序遍历是现房问节点的后代，在访问节点本身，后序遍历应用是计算一个目录和他子目录中所有文件占用空间的大小。

```JavaScript
class PostOrderTraverseNode extends BinarySearchTree {
   postOrderTraverse (root,callback) {
     this.postOrderTraverseNode(root,callback)
   }
  postOrderTraverseNode(node, callback) {
    console.log(node)
    if(node !== null) {
      this.postOrderTraverseNode(node.left,callback);
      this.postOrderTraverseNode(node.right,callback)
      callback(node.key)
    }
  }
}
const b = new PostOrderTraverseNode()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)

b.postOrderTraverse(tree.root, (key) => console.log(key))
```

### 搜索树中的值

在树中搜索经常有三种搜索：

1. 最小值

2. 最大值

3. 特定值

### 搜索最小值和最大值

根据树的规定，那么最小值一定在最左端的末枝，对应的最大的在最右端。

先实现最小值方法

```JavaScript
class MidNode extends BinarySearchTree {
  mid (root) {
    return this.midNode(root)
  }
  midNode (node) {
    if(node) {
      while(node && node.left !== null) {
        node = node.left
      }
      return node.key
    }
    return null
  }
}

const midNode = new MidNode()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
console.log(midNode.mid(tree.root)) // 3
```

同理最大值就是最右了

```JavaScript
class MaxNode extends BinarySearchTree {
  max (root) {
    return this.maxNode(root)
  }
  maxNode (node) {
    if(node) {
      while(node && node.right !== null) {
        node = node.right
      }
      return node.key
    }
    return null
  }
}

const midNode = new MaxNode()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
console.log(midNode.max(tree.root))// 15
```

### 搜索特定值

在BST树中实现搜索

```JavaScript

class Search extends BinarySearchTree {
  search (root, key) {
    return this.searchNode(root, key)
  }
  searchNode (node, key) {
    if (node === null) {
      return false;
    }
    
    if (key < node.key) { // 小于向左
      return this.serachNode(node.left, key);
    } else if (key > node.key) {// 大于向右
      return this.searchNode(node.right, key)
    } else {// 不大于不小于就是等于
      return true
    }
  }
}
const search = new Search()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
console.log(search.search(tree.root, 7)) // true
```

## 移除一个节点

移除一个节点比较复杂，因为移除有很多情况。

```JavaScript
class Remove extends BinarySearchTree {
   mid (root) {
    return this.midNode(root)
  }
  midNode (node) {
    if(node) {
      while(node && node.left !== null) {
        node = node.left
      }
      return node.key
    }
    return null
  }
  remove (root,key) {
    this.root = this.removeNode(root, key)
  }
  removeNode (node, key) {
    if (node === null) {
      // 如果节点为空
      return null
    }

    if (key < node.key) {
      // 递归左节点
      node.left = this.removeNode(node.left, key)
      // 返回值
      return node
    } else if (key > node.key) {
      // 递归右节点
      node.right = this.removeNode(node.right,key)
      // 返回值
      return node
    } else {
        // 不大于不小于那么就是等于

      // 第一种情况它是最末尾的一个叶节点，没有子节点了

      if (node.left === null && node.rhght === null) {
        // 移除当前节点的引用
        node = null;
        return node;
      }

      // 第二种情况不是叶节点但是只有一个子节点

      if (node.left === null) {
        node = node.right;
        return node
      } else if (node.right === null) {
        node = node.left;
        return node;
      }

      // 第三种情况它有两个子节点
      // 思路是使用左叶或者右叶替换掉要被移除的节点

      const aux = this.mid(node.right)
      node.key = aux.key
      node.right = this.removeNode(node.right, aux.kye)
      return node
    }
  }
}
const remove = new Remove()
const tree = new BinarySearchTree()
tree.insert(7)
tree.insert(15)
tree.insert(5)
tree.insert(3)
console.log(tree.root)
remove.remove(tree.root, 7)
console.log(tree.root)
```

# 图

> 图是另一种非线性结构，而且图是一个庞大的主题，深入探索图的奇妙世界每个部分都可以写一本书

这一章右很多图，我画不出来。

## 图的相关术语

`图`是网格结构的抽象模型，图是右一组由边链接的节点（或定点），学习图是十分重要的，因为任何二元关系都可以用图来表示

任何的社交网络如Facebook 、 微博、 知乎都可以用图来表示

图在数学和技术上的基础

一个图 G = (V,E) 由以下元素组成

V: 一组定点

E: 一组边，链接V中的点

在着手实现算法之前，先了解一些术语

由一条边链接在一起的顶点称为相邻顶点

一个顶点的度是其相邻顶点的数量

路径是顶点v1，v2，v3...vk的一个连续序列，其中vi好vi+1是相邻的

简单路径要求不包含重复的顶点，环也是一个简单路径。

如果图中不存在环，则称该图是无环的，如果图中每两个顶点中都存在路径，那么该图是连通的。

## 有向图和无向图

图可以是无向的（边没有方向）或是有向的（边是右方向的），有向图的边是有一个方向。

如果图中每两个顶点间在双向都存在路径，则该图是强连通的。

图还可以是未加权的或者是加权的

可以使用图来解决计算机科学世界中的很多问题，比如所搜图中的一个特定顶点或搜索特定边，寻找图中一条路径（从一个顶点到另一个顶点），寻找两个顶点之间的最短路径，以及环检测。

## 图的表示

从数据结构的角度来说，右很多种方式来表示图，在所有的表示法中，不存在绝对正确的方式，图的正确表示法取决于待解决的问题和图的类型

## 邻接矩阵

图最常见的实现是相邻矩阵，每个节点都和一个整数相关联，该整数将作为数组的索引，我们用一个二维数组来表示顶点之间的链接，如果索引为i的节点和索引为j的节点相邻，则array[i][j] === 1， 否则array[i][j]===0.

不是强连通的图（稀疏图）如果用邻接矩阵来表示，则矩阵中将会用很多0，这意味着浪费了计算机储存空间来表示根本不存在的边，例如找给定顶点的相邻顶点，及时该顶点只有一个相邻顶点，也不得不迭代一整行，相邻矩阵表示法不好的另一个理由是图中数量可能会边，二维数组不太灵活。

## 邻接表

另一种表示图的数据结构叫做邻接表，邻接表由途中每个顶点的相邻列表组成，存在好几种方式来实现这种结构，可以使用列表，链表，甚至是散列表或者字典来表示相邻顶点列表。

尽管邻接表可能对大多数问题来说都是更好的选择，但以上两种表示方法都很有用，且他们有着不同的性质，所以在本书中将使用邻接表表示法。

## 创建图类

```JavaScript
class Graph {
    constructor {
        this.vertices = []
        this.adjList = new Map() // 使用Map数据结构来储存。
    }
}
```

使用一个数组来储存所有顶点的名字，以及一个字典来储存邻接表，字典将会使用顶点的名字作为键，邻接顶点列表作为值。

```JavaScript
class Graph {
    constructor() {
        this.vertices = []
        this.adjList = new Map() // 使用Map数据结构来储存。
    }

    // 向图中新增一个新的顶点。
    addVertex (v) {
    this.vertices.push(v);
    this.adjList.set(v,[]);
    }

    //添加边
    addEdge (v, w) {
    // 给顶点v添加一条到w的边
    this.adjList.get(v).push(w)
    // 相反给顶点w添加一天到v的边
    this.adjList.get(w).push(v)
    }
}
const graph = new Graph()
const myVerties = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']

for (let i = 0; i < myVerties.length; i++) {
graph.addVertex(myVerties[i])
}
console.log(graph) // 如果不好理解的话跑一下这段代码看看这两个console
graph.addEdge('A','B')

console.log(graph)
```

## 表的遍历

和树的数据结构类似，也可以访问图的所有节点，有两种算法可以对图进行遍历，`广度优先搜索`和`深度优先搜索`，图遍历可以用来寻找特定的顶点或者寻找两个顶点之间的路径，检查图是否连通是否有环等。

先理解一下图遍历的思想方法。

图遍历算法的思想是必须追踪每个第一次访问的节点，并且追踪有那些节点还没有完全探索，对于良好总图遍历算法，都需要明确指出第一个被访问的顶点。

完全探索一个顶点要求我们查看该顶点的每一条边，对于每一条边所链接的没有被访问过的顶点，将其标注为发现，并将其加进待访问顶点列表中。

为了保证算法的效率，务必访问每个顶点两次，连通图中每条边和顶点都会被访问到。

广度优先搜索算法和深度优先搜索算法基本上是相同的，只有一点不同，那就是待访问顶点列表的数据结构

深度优先使用栈，通过将顶点存入占中，顶点是沿着路径被弹错的，存在的新的 邻顶点就过去访问。

广度优先使用队列，通过将顶点存入队列中，先进入队列的顶点先被探索。

当要标注已经被访问过的顶点时候使用三种颜色来翻译他们的状态

白色，表示该顶点还没有被访问过

灰色，表示该顶点被访问过，但是并未被探索

黑色，表示该顶点被访问过且被完全探索过

这就是之前提到的务必访问每个顶点最多两次的原因。

## 广度优先算法

广度优先搜索算法会从指定的第一个顶点开始遍历图，先访问其所有的相邻节点，就像一次访问图的一层，换句话说就是先宽后深的访问顶点。

以下是从顶点v开始广度搜索算法所遵循的步骤

1. 先创建一个列队Q

2. 将v标注为被发现灰色，并将v加入队列Q

3. 如果Q非空，则运行以下步骤：
    1. 将u从Q中出队列
    2. 将标注为U为被发现的灰色
    3. 将U所有未被访问过的邻节点白色加入列队Ｑ
    4. 将Ｕ标注为已被探索的黑色

文字看不太懂，先看看代码如何实现的吧

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

class Graph {
    constructor() {
        this.vertices = []
        this.adjList = new Map() // 使用Map数据结构来储存。
    }

    // 向图中新增一个新的顶点。
    addVertex (v) {
    this.vertices.push(v);
    this.adjList.set(v,[]);
    }

    //添加边
    addEdge (v, w) {
    // 给顶点v添加一条到w的边
    this.adjList.get(v).push(w)
    // 相反给顶点w添加一天到v的边
    this.adjList.get(w).push(v)
    }

    initializeColor () {
      const color = []
      for (let i = 0; i < this.vertices.length; i++) {
        color[this.vertices[i]] = 'white';
      }
      return color;
    }

    bfs (v, callback) {
        let color = this.initializeColor(); // 将所有顶点渲染为白色
      const queue = new Queue(); // 生成队列
      queue.enqueue(v);	 // 添加入队

      while (!queue.isEmpty()) { // 如果列队不为空
        let u = queue.dequeue(); // 从列队中出队第一个顶点
        let neighbors = this.adjList.get(u);//取得这个顶点包含其所有林甸的邻接表

       color[u] = 'grey'; // 标记顶点被访问过，但是没有被探索过

       for (let i = 0; i < neighbors.length; i++) {
    // 访问这个顶点的边
          const w = neighbors[i];
          if (color[w] === 'white') { // 如果它还没有被访问过则将他标记为已访问
            color[w] = 'grey';
            queue.enqueue(w); // 并将这个顶点加入队列中
          }
       }
       color[u] = 'black'; // 当完成探索该顶点和相邻顶点后将其标注为已探索的黑色
       if (callback) {// 判断有没有回调如果右执行回调
       callback(u)
       }
      }
    }
}
const graph = new Graph()
const myVerties = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']

for (let i = 0; i < myVerties.length; i++) {
graph.addVertex(myVerties[i])
}
graph.addEdge('A','B')
graph.addEdge('A','C')
graph.addEdge('A','D')
graph.addEdge('C','D')
graph.addEdge('C','G')
graph.addEdge('D','G')
graph.addEdge('D','H')
graph.addEdge('B','E')
graph.addEdge('B','F')
graph.addEdge('E','I')
function printNode(value) {
console.log('访问了顶点', value)
}
graph.bfs(myVerties[0],printNode)

// 一知半解的感觉，代码实现上没有错误，感觉反正让我写我写不出来
```

### 使用BFS寻找最短路径

使用BFS搜索来解决一个问题

给定一个图的G和源顶点V，找出对每个顶点U，U和V之间的最短路径的距离，以边的数量多少来衡量。

对于给定顶点V，广度优先搜索hUI访问所有与其距离为1的顶点，接着是距离为2的顶点，以此类推，所以可以使用广度优先算法来解决这个问题，修改bfs方法返回给我们一些信息：
从V到U的距离d[U]
前溯点pred[U]，用来推到出从V到其他每个顶点U的最短路径

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

class Graph {
    constructor() {
        this.vertices = []
        this.adjList = new Map() // 使用Map数据结构来储存。
    }

    // 向图中新增一个新的顶点。
    addVertex (v) {
    this.vertices.push(v);
    this.adjList.set(v,[]);
    }

    //添加边
    addEdge (v, w) {
    // 给顶点v添加一条到w的边
    this.adjList.get(v).push(w)
    // 相反给顶点w添加一天到v的边
    this.adjList.get(w).push(v)
    }
    
    initializeColor () {
    	const color = []
      for (let i = 0; i < this.vertices.length; i++) {
      	color[this.vertices[i]] = 'white';
      }
      return color;
    }
    
    bfs (v, callback) {
    	let color = this.initializeColor()
      const queue = new Queue()
      let d = [], pred = [];
      queue.enqueue(v);
      
      for(let i = 0; i < this.vertices.length; i ++) {
      d[this.vertices[i]] = 0;
      pred[this.vertices[i]] = null;
      }
      
      while(!queue.isEmpty()) {
      const u =  queue.dequeue();
      let neighbors = this.adjList.get(u)
      color[u] = 'grey'
      for (let i =0; i < neighbors.length; i++) {
     		const w = neighbors[i]
        if (color[w] === 'white') {
        	color[w] = 'grey'
          d[w] = d[u] + 1;
          pred[w] = u
          queue.enqueue(w)
        }
      }
      color[u] = 'black'
      }
      return {
      distances: d,
      predecessors: pred
      }
    }
}
const graph = new Graph()
const myVerties = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']

for (let i = 0; i < myVerties.length; i++) {
graph.addVertex(myVerties[i])
}
graph.addEdge('A','B')
graph.addEdge('A','C')
graph.addEdge('A','D')
graph.addEdge('C','D')
graph.addEdge('C','G')
graph.addEdge('D','G')
graph.addEdge('D','H')
graph.addEdge('B','E')
graph.addEdge('B','F')
graph.addEdge('E','I')
console.log(graph.bfs(myVerties[0]))
```

深入学习最短路径算法

上面的图是不加权的，如果要在加权图中寻找最短路径，例如城市A到城市B之间的最短路径，那么广度优先搜索未必合适。

有很多专用其他的算法，来对应不同情况下最短路径问题，正如前面所说的，图是一个广泛的主题，单单一个最短路径问题和他的变种问题就右很多很多解决方案，但是在学习这些方案之前，需要很好的掌握图的基本概念，从而更轻松的学习其他解决方案。

### 深度优先搜索

深度优先搜索算法将会从第一个指定的顶点开始遍历图，沿着路径一直到这条路径最后一个顶点被访问，接着按原路回退探索下一条路径，换句话说，他是先深度后广度的访问顶点。

深度优先搜索算法不需要一个源顶点，在深度优先算法中，若图中顶点V未被访问，则访问该顶点V，要访问顶点V，需要按照一下步骤进行。

1. 标注V为未发现的灰色

2. 对于V的所有未访问的邻点W都进行一次访问

3. 将V标记为已探索

深度优先搜索的步骤是递归，这意味着深度优先搜索算法使用栈来储存函数调用，由递归函数调用所创建的栈。

```JavaScript
    constructor() {
        this.vertices = []
         this.adjList = new Map() // 使用Map数据结构来储存。
    }

    // 向图中新增一个新的顶点。
    addVertex (v) {
    this.vertices.push(v);
    this.adjList.set(v,[]);
    }

    //添加边
    addEdge (v, w) {
    // 给顶点v添加一条到w的边
    this.adjList.get(v).push(w)
    // 相反给顶点w添加一天到v的边
    this.adjList.get(w).push(v)
    }
    dfsVisit (u, color, callback) {
    	color[u] = 'grey';
      if (callback) {
      	callback(u)
      }
      const neighbors = this.adjList.get(u)
      for (let i = 0; i < neighbors.length; i++) {
      const w = neighbors[i]
      if (color[w] === 'white') {
      	this.dfsVisit(w,color,callback);
      }
      }
      color[u] = 'black';
    }
    initializeColor () {
    	const color = []
      for (let i = 0; i < this.vertices.length; i++) {
      	color[this.vertices[i]] = 'white';
      }
      return color;
    }
    
    dfs (callback) {
    	let color = this.initializeColor();
      for (let i = 0; i < this.vertices.length; i ++) {
      	if (color[this.vertices[i]] === 'white'){
        this.dfsVisit(this.vertices[i],color,callback)
        }
      }
    }
}
const graph = new Graph()
const myVerties = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']

for (let i = 0; i < myVerties.length; i++) {
graph.addVertex(myVerties[i])
}
graph.addEdge('A','B')
graph.addEdge('A','C')
graph.addEdge('A','D')
graph.addEdge('C','D')
graph.addEdge('C','G')
graph.addEdge('D','G')
graph.addEdge('D','H')
graph.addEdge('B','E')
graph.addEdge('B','F')
graph.addEdge('E','I')
function printNode(value) {
console.log('访问了顶点', value)
}
console.log(graph)
console.log(graph.dfs(printNode))
```

# 排序和搜索算法

在日常生活中需要值信息，比如春村在数据结构里面的信息，排序和搜索算法广泛的运用在解决日常生活问题中。

## 排序算法

从最简单的开始

先实现一个用来表示待排序和搜索的数据结构

```JavaScript
class ArrayList {
    constructor () {
        this.array = []
    }
    insert (item) {
        array.push(item)
    }
    toString () {
        return array.join()
    }
}
```

上面实现了一个非常简单的数据结构他将项储存在数组中，并且写了一个方法向数据结构中添加元素，为了帮助验证结果结果重写了toSting方法。

### 冒泡排序

冒泡排序是所有排序算法中最简单的，然而从时间复杂度来看他是最差的。

冒泡排序比较任何两个相邻的项，如果第一个比第二个大，则交换他们，元素向上移动到正确的位置，好像气泡升至表面一样，冒泡排序因此得名。

实现以下冒泡排序

```JavaScript
class ArrayList {
    constructor () {
        this.array = []
    }
    insert (item) {
        this.array.push(item)
    }
    toString () {
        return this.array.join()
    }
    swap(index1, index2) {
    // 交换数组的两个元素
    	const aux = this.array[index1]
      this.array[index1] = this.array[index2]
      this.array[index2] = aux
    }
    
    // 冒泡排序
    bubbleSort () {
    	const length = this.array.length
      for (let i = 0; i < length; i++) {
      	for (let j = 0; j < length - 1; j++) {
        	if (this.array[j] > this.array[j+1]) {
          	this.swap(j, j+1)
          }
        }
      }
    }
}

// 测试排序代码
// 逆序创建一个ArrayList
function createNonSortedArray(size) {
	const arr = new ArrayList()
  for (let i = size; i > 0; i --) {
  	arr.insert(i)
  }
  return arr
}
const arr = createNonSortedArray(100)
console.log(arr.toString()) // 确定为逆序
arr.bubbleSort()
console.log(arr.toString()) // 确定排序完毕

```

### 选择排序

```JavaScript
class ArrayList {
    constructor () {
        this.array = []
    }
    insert (item) {
        this.array.push(item)
    }
    toString () {
        return this.array.join()
    }
    swap(index1, index2) {
    // 交换数组的两个元素
    	const aux = this.array[index1]
      this.array[index1] = this.array[index2]
      this.array[index2] = aux
    }
    
   selectionSort () {
    	const length = this.array.length
    	let indexMin
    	for (let i = 0; i < length - 1; i++) {
      	indexMin = i;
        for (var j = i; j < length; j++) {
        	if (this.array[indexMin]>this.array[j])
          	indexMin = j
        }
        if (i !== indexMin) {
        	this.swap(i,indexMin)
        }
      }
    }
}

// 测试排序代码
// 逆序创建一个ArrayList
function createNonSortedArray(size) {
	const arr = new ArrayList()
  for (let i = size; i > 0; i --) {
  	arr.insert(i)
  }
  return arr
}
const arr = createNonSortedArray(100)
console.log(arr.toString()) // 确定为逆序
arr.selectionSort()
console.log(arr.toString()) // 确定排序完毕

```

这两段排序的时间复杂度都是O(n²)，他们都有两个嵌套循环，这导致了二次方的复杂度。

### 插入排序

插入排序每次排一个数组项，以此方式构建最后的排序数组，假设第一项已经排序了，接着他和第二项进行比较，第二项是应该待在原位还是插到第一项之前，这样头两项就已正确排序，接着比较第三项。判断他应该在哪里。

```JavaScript
class ArrayList {
    constructor () {
        this.array = []
    }
    insert (item) {
        this.array.push(item)
    }
    toString () {
        return this.array.join()
    }
    swap(index1, index2) {
    // 交换数组的两个元素
    	const aux = this.array[index1]
      this.array[index1] = this.array[index2]
      this.array[index2] = aux
    }
    
   insertionSort () {
   const length = this.array.length
   let j, temp
   
   for(let i = 1; i < length; i++) {
   	j = i
    temp = this.array[i]
    while(j > 0 && this.array[j-1] > temp) {
    	this.array[j] = this.array[j-1]
      j--
    }
    this.array[j] = temp
   }
   }
}

// 测试排序代码
// 逆序创建一个ArrayList
function createNonSortedArray(size) {
	const arr = new ArrayList()
  for (let i = size; i > 0; i --) {
  	arr.insert(i)
  }
  return arr
}
const arr = createNonSortedArray(100)
console.log(arr.toString()) // 确定为逆序
arr.insertionSort()
console.log(arr.toString()) // 确定排序完毕
```

在排序小型数组时候，其效率高于冒泡排序

### 并归排序

并归排序是一个可以被实际使用的排序算法，并归排序的算法复杂度为O(n log n次方)

JavaScript的Array定义了一个sort函数，用于培训JavaScript数组，但是ECMASsript 并没有规定使用那种算法进行排序，个浏览器厂商可以自行实现算法，那么火狐使用了并归排序，谷歌则使用了排序排序。

并归排序是一种分治算法，其思想是将原始数组分割成较小的数组，直到每个小数组只有一个位置，接着将小数组合并成大数组，最后只有一个排序完毕的大数组。

```JavaScript
class ArrayList {
    constructor () {
        this.array = []
    }
    insert (item) {
        this.array.push(item)
    }
    toString () {
        return this.array.join()
    }
   
   // 并归函数入口
   mergeSort () {
   	this.array = this.mergeSortRec(this.array)
   }
   
   // 将数组递归拆分成只有一个元素的数组
   mergeSortRec (array) {
   		const length = array.length
      if (length === 1) {// 递归函数终止条件
      	return array
      }
      
      const mid = Math.floor(length / 2);  // 取得中间值
      const left = array.slice(0,mid), // 左切片
      			right = array.slice(mid, length); // 右切片
      return this.merge(this.mergeSortRec(left), this.mergeSortRec(right)) // 递归调用。
   }
   
   merge(left, right) {
    console.log(left, right) // 如果理解不了的话可以看控制台打印出来的内容然后带入下面的迭代就比较好懂了。
   	const result = [] // 声明一个数组用来储存归并过程中的新数组
  	let il = 0, ir = 0;// 两个用于迭代的变量
    while(il < left.length && ir < right.length) { // 迭代两个数组
    	if (left[il] < right[ir]) { // 对比左边是否小于右边
      	result.push(left[il++]); // 如果是那么将左边添加到数组
      } else {
      	result.push(right[ir++]); // 如果不是则将右边添加到数组
      }
    }
    while (il < left.length) { // 接下来将左边数组剩余项添加到归并数组中
    	result.push(left[il++])
    }
    while (ir < right.length) { // 将右边数组生育项添加到归并数组中
    	result.push(right[ir++])
    }
     return result
   }
}

// 测试排序代码
// 逆序创建一个ArrayList
const arr = new ArrayList()
arr.insert(4)
arr.insert(2)
arr.insert(8)
arr.insert(3)
arr.insert(5)
arr.insert(1)
arr.insert(7)
arr.insert(6)
console.log(arr.toString()) // 确定为逆序
arr.mergeSort()
console.log(arr.toString()) // 确定排序完毕
```

这段代码理解起来比较复杂，首先对递归有了新的认识，从运算结果推断过程，还有就是发明这个算法的人真的好聪明。

学习这段代码的时候我是一部一部console打印过来的。我在代码中留下了其中最重要的一个console，根据console的内容来带入运行，就可以比较轻松地理解排序过程了。

