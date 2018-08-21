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
