---
title: JavaScript中的数组
date: 2018-03-26 20:02:33
tags: JavaScript
---

JavaScript 对数组的操作有一下几种，记录下防止自己过快的忘记

<!-- more -->

# 遍历

## filter()

特点：不修改原始数组，而是返回一个新的数组，其中包含的是通过提供函数实现的所有元素

语法：const newArray = arr.filter(callback[, thisArg])
分别是必选回调函数和可选执行回调函数的 this 的值

具体描述：filter 为数组中的每个元素都调用一次回调函数，并且利用所有使得回调函数返回`经过转换后所有为值为真`的元素创建一个新的数组
会从头遍历到数组结束，filter 遍历元素范围在第一次调用回调函数之前就确定了，所有在 filter 之后被添加的元素不会被遍历到。没有主动终止遍历的操作

## map()

特点：不修改原始数组，返回一个新的数组，其中结果是数组中的每个元素都调用一次提供的回调函数后返回的结果

语法：newArray = arr.map(callback(item, index, array){
// 返回的数组元素
}[,thisArg])

分别是必选回调函数使用三个参数 item 数组中正在处理的当前元素，index 第二个参数，数组中正在处理的当前元素索引，array，map 方法被调用的数组，thisAry 执行回调使用的 this 值。

map 方法在 react 用的很多，因为 react 可以渲染数组里面的组件，只需要给每个元素添加一个独有的 key 即可，尽量不要使用 index 作为 key，可扩展性很差。

描述：map 方法会给原始数组的每个元素按`顺序`调用一次回调函数，回调函数每次返回值组成一个包括 undefined 组合成新数组返回。没有主动结束遍历的方法。

同样的 map 方法处理数组的时候在第一次调用回调之前就确定了。

## some()

some 方法测试数组中的某些元素是否通过提供的测试函数
特点：只返回一个布尔值
语法：arr.some(cllback[,thisArg])
参数分别为用于测试的回调函数和回调函数执行时 this 的值

描述：some 为数组每个元素执行一次回调函数，直到找到一个`真值`或`经过转换后所有为值为真`如果找到这样一个值，则立刻返回 true，如果找不到就返回 false

同样的 some 函数在第一次调用回调之前就确定了范围。

## every()

every 方法测试数组中所有的元素是否通过了提供的测试函数
特点：只返回一个布尔值
语法：arr.every(callback[,thisArg])
参数分别为用于测试的回调函数和回调函数执行时的 this 值

描述：every 方法为数组中的每个元素都执行一次回调函数，直到他找到一个使回调函数返回`假值`或`经过转换后所有值为假`，否则 every 方法将返回 true，无法主动终止。

every 当所有的元素都符合才返回 true，另外空数组也是返回 true(空数组中的所有元素都符合既定的元素，因为空数组没有元素)

## forEach()

forEach 方法对数组的每一个元素都执行一次提供的函数
特点：没有返回值
语法：arr.forEach(callback,this)
参数分别为回调函数和执行时的 this 值

描述：forEach 方法按升序为数组中含有值每一项执行一次回调函数，没有返回值，没有新数组

同样的没有方法跳出 forEach 循环，除非抛出一个异常。但是这样操作 forEach 是错误的，因为完全可以使用一个简单的循环作为替代。

## lashIndexOf()

lashIndexOf 方法返回指定的元素在数组中的最后一个索引如果不存在则返回-1，从数组开始到结束即从 0 开始

特点：返回指定元素在数组中的最后一个索引，比如[2,3,3,3,3,2].lastIndexOf(2)返回 5
语法：arr.lastIndexOf(searchElement[,formIndex = arr.length -1])

参数分别为要被查找的元素，或者提供第二个参数逆向查找

返回数组中最后一个指定元素的索引

描述：lastIndexOf 使用严格相等即===来进行对比

## indexOf()

indexOf 方法返回数组中指定元素的第一个索引，如果不存在则返回-1

语法：arr.indexOf(searchElement)
要搜索的元素

返回收个在数组中找到的指定值索引，如果不存在则返回-1

描述：indexOf 和 lastIndexOf 一样使用===进行对比

## for of

for of 语句在可迭代对象包括（Array、Map、Set、String、TypedArray、arguments）上创建一个迭代循环，调用自定义迭代钩子，为每个不同的属性执行语句

语法：for(let variable of iterable)
variable
在每次迭代中，将不同属性的值分配给变量。
iterable
被迭代枚举其属性的对象。

关闭迭代器可以用 break continue throw 或者 return 终止迭代

for in 和 for of 区别

两者都是用于迭代一些东西，他们之间的主要区别在于他们迭代的方式

for in 语句以原始插入顺序迭代对象的可枚举属性

for of 语句用于遍历可迭代对象定义的要迭代的数据
