---
title: JavaScript中Sort函数
date: 2017-12-12 18:51:15
tags: JavaScript
---
由于昨天看了几个排序，又想到JavaScript常用`Array.sort`进行排序，所以搜索了一下JavaScript中sort函数使用了那种排序方法。

> sort() 方法在适当的位置对数组进行排序，并返回数组，同时sort排序不一定是稳定的 -- MDN文档

语法

> arr.sort()

> arr.sort(compareFunction)

sort()函数默认根据字符串Unicode码点进行排序

比如

> arr.sort()

```
var a = ['cherries', 'apples', 'bananas']

a.sort(); //['apples', 'bananas', 'cherries']

var b = [1, 10, 21, 2];

b.sort(); //[1, 10, 2, 21]
// 因为在Unicode指针顺序中'10'在'2'之前

```

这种这种按Unicode顺序排序一般不符合使用要求。 所以大多都会使用

> arr.sort(compareFunction)

给sort函数传一个比较函数

如果指明了 compareFunction ，那么数组会按照调用该函数的返回值排序。即 a 和 b 是两个将要被比较的元素：

如果 compareFunction(a, b) 小于 0 ，那么 a 会被排列到 b 之前； 如果 compareFunction(a, b) 等于 0 ， a 和 b 的相对位置不变。备注： ECMAScript 标准并不保证这一行为，而且也不是所有浏览器都会遵守（例如 Mozilla 在 2003 年之前的版本）； 如果 compareFunction(a, b) 大于 0 ， b 会被排列到 a 之前。 compareFunction(a, b) 必须总是对相同的输入返回相同的比较结果，否则排序的结果将是不确定的。

谷歌搜了一下在谷歌浏览器的v8引擎中 sort只有两种方法 对于长度小于等于10的数组使用插入排序， 长度大于10的数组使用快速排序。 具体看[V8源码](https://github.com/v8/v8/blob/be3c2cdd8de464dd0832c0ba4c9159ce5a0ce979/src/js/array.js#L707)

我看不懂

看了很多博客介绍

插入排序

```

function insertionSort(arr) {
    // 声明一个函数函数接受一个数组作为参数
    for (var i = 1; i < arr.length; i++) {
    // 使用for进行循环
        var element = arr[i];
        // 储存当前循环的值
        for (var j = i - 1; j >= 0; j--) {
        // 嵌套循环，获取arr[i]的前一个
            var tmp = arr[j];
            var order = tmp - element;
            // 对比前一个和当前的值
            if (order > 0) {
                // 如果前一位大于后一位
                arr[j + 1] = tmp;
                // 将前一位放到后一位的位置
            } else {
                break;
            }
        }
        arr[j + 1] = element;
    }
    return arr;
}

var arr = [6, 5, 4, 3, 2, 1];
console.log(insertionSort(arr));
```
