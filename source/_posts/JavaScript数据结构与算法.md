title: JavaScript算法与数据结构
date: 2018-8-20 22:26:38
tags: JavaScript
---

认真学习JavaScript数据结构

<!-- more -->

# 准备工作

**比较器**

一个比较器类拥有以下方法:

```typescript
  // 类接收一个可选的比较函数作为参数
  compare: (a, b) => number
  // 判断两个参数是否相等
  equal: (a: (number | string), b: (number | string)) => boolean
  // 判断a是否小于b
  lessThan: (a: (number | string), b: (number | string)) => boolean
  // 判断a是否大于b
  greaterThan: (a: (number | string), b: (number | string)) => boolean
  // 判断a是否小于等于b
  lessThanOrEqual: (a: (number | string), b: (number | string)) => boolean
  // 判断a是否大于等于b
  greaterThanOrEqual: (a: (number | string), b: (number | string)) => boolean
  // 反转比较函数的两个参数即从a,b转换为b,a
  reverse: () => void

```

构造一个这样的比较器函数之后可以在后面的数据结构中进行使用


# 链表

在计算机科学中,链表是数据元素的线性集合,其中每个元素的顺序并不是由它们在内存中的物理位置给出的,而是每个元素都持有下一个元素的指针,它是一组节点组成的数据结构,这些节点一起表示序列,在最简单的形式下,每个节点由`数据`和`到序列中下一个节点的引用`组成的,这种结构允许在迭代的时候在任意位置有效的插入或者移除元素,更复杂的变体添加了额外的连接,允许有效插入或从任意元素引用删除,列表的缺点是访问时间是线性的(O(n),并且无法优化),例如数组的随机访问,在链表中是不可行的,与链表相比,数组局域更好的缓存局部性.

![Linked List](https://upload.wikimedia.org/wikipedia/commons/6/6d/Singly-linked-list.svg)

其支持一下几种主要的方法:

- 插入

```javascript

/**
 * 这个插入节点非常有趣
 */
  append(value: any): LinkedListInterface {

    // 首先根据参数生成一个节点这里假设 value = 1
    // 那么生成的节点就是 {value:1, next: null}
    const newNode = new LinkedListNode(value)
    // 假如一开始链表是空的
    if (!this.head) {
      // 那么头节点和尾节点指向的是同一个对象 this.head = {value:1, next: null}
      // this.tail = {value: 1, next: null} 注意这里引用的是同一个对象
      this.head = newNode
      this.tail = newNode
      return this
    }


    // 那么第二次添加节点链表头结点不为空执行下面的赋值 这里假设 value参数为2
    // 首先this.tail.netx = {value:2,next:null}
    // 那么this.tail 完整储存的内容就是 {value:1, next: {value:2,next:null}}
    // 因为this.head 和 this.tail 指向的是同一个对象所以完整的的链表是这样的
    // this.head的值  {value:1, next: {value:2,next:null}}
    // this.tail的值   {value:1, next: {value:2,next:null}}
    this.tail.next = newNode

    // 接下来将tali 赋值为 {value: 2, next:null}
    // 此时链表结构为
    // this.head = {value:1, next: {value:2,next:null}}
    // this.tail = {value: 2, next:null}
    // 这里this.head.next引用的对象和tail引用的对象是同一个对象
    // 所以下次赋值会重复这个过程,即最后一个节点和tail节点引用的是同一个对象
    // 所以重复
    // this.tail.next = newNode
    // this.tail = newNode
    // 即可完成添加新的节点
    this.tail = newNode

    return this

  }

```


- 搜索
- 删除
- 反转
- 反向遍历
  
## 复杂度

时间复杂度

| 类型 | 访问 | 搜索 | 插入 | 删除 |
| ---- | ---- | ---- | ---- | ---- |
| 列表 | O(n) | O(n) | O(1) | O(1) |
| 数组 | O(1) | O(n) | O(n) | O(n) |

## 空间复杂度

O(n)


# 双向链表

在计算机科学中,双向链表是一种线性数据结构,由一组被称为节点的顺序连接记录组成,每个节点包含两个字段,称为链接,节点中有上一个节点和下一个节点的引用,可以被概念化为由两个相同数据形成的两个链表,但是以相反的顺序组成

![Doubly Linked List](https://upload.wikimedia.org/wikipedia/commons/5/5e/Doubly-linked-list.svg)

# 构成双向链表的基本方法

- insert
- delte
- reverse traversal

# 相关复杂度

| Access(访问) | Search | insertin | deleteion |
| ------------ | ------ | -------- | --------- |
| O(n)         | O(n)   | O(1)     | O(1)      |

## 空间复杂度

O(n)

