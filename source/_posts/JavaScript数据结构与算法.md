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
