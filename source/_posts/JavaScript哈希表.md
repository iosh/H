---
title: JavaScript字典和哈希表
date: 2017-12-20 19:21:54
tags: JavaScript
---
##  字典
>  字典是一种以键 - 值对应形势储存的数据结构，就像手机里面的电话本一样，只需要记录名字和对应的手机号码，下次拨打的时候只需要查找名字就可以了。这里键就是手机号的名字，而真正拨打的手机号是查找到的值。

在ES6中新增了一个原生的Map数据结构，参考ES6的Map数据结构实现一下。（JavaScript也在发展）
<!-- more -->
1. 建立字典类
```javascript
class Dictionary{
	constructor(){
		this.items = {};
	};
	set(key, value){ // 新增一个键值
		this.items[key] = value;
	};
	remove(key){ // 删除一个键值
		if(this.has(key)){
			delete this.items[key];
			return '删除成功';
		}
		return '删除失败';
	};
	has(key){ // 判断key是狗存在
		return this.items.hasOwnProperty(key); // 返回一个布尔值
	};
	get(key){ // 读取一个值
		return this.has(key) ? this.items[key] : undefinded
	};
	clear(){ // 清除字典
		this.items = {};
	};
	size(){ // 返回长度
		return Object.keys(this.items).length
	};
	keys(){ // 返回字典的键
		return Object.keys(this.items);
	};
	values(){ // 返回字典所有的值
		let value = [];
		for(let i in this.items){
			if(this.has(i)){
				value.push(this.items[i])
			}
		}
		return value;
	};
}

const b = new Dictionary()
b.set(1,1)
b.set(2,2)
b.set(3,3)
b.set(4,4)
b.set(5,5) // 添加五个键对
b.get(5) // 5
b.size() // 5
b.values() //[1, 2, 3, 4, 5]
b,has(5) // true
b.remove(1) // 删除成功
b.values() // [2, 3, 4, 5]
```




>  哈希表也叫散列表，是根据键(key)而直接访问在内存存储位置的数据结构，也就是说，他通过计算一个关于键值的函数，将所需查询的数据映射到表中一个位置来访问，加快了查找速度，这个映射函数称作散列函数，存放记录的数组称作散列表   ---维基百科


ES6新增的Map数据结构很适合用于哈希表，JavaScript的对象Object，本质上是键值对的结合（Hash结构），但是传统上只能用字符串当做键，这给他的使用带来了很大的限制。
```JS
const a = {};
a[1] = '233'
a['1'] // 233
```
这里的数字1被转换成了字符串1，本意是将一个数字1传入对象，但是Object只接受字符串所以被自动转换了。

为了解决这个问题，ES6提供了Map数据结构，他类似于对象，也是键值对的集合，但是*键*的范围不限于字符串，各种类型的值（包括对象）都可以当做键，也就是说Object提供了*字符串 - 值*的对应，而Map结构提供了*值 - 值*的对应，是一种更加完善的Hash结构实现 --- 阮一峰ES6

用class实现一个表
[charCodeAt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)用法
```js
class HashTable{
constructor(){
		this.table = [];
	};
	getHashTableCode(key){ // 获取随机hash值
		let Hash = 0;
		for(let i = 0; i < key.length; i++) {
			// aharCodeAt 会回传置顶字符串的Unicode编码，所以可以包含中文
			Hash += key.charCodeAt(i);	
		}
		  // 为了取到较小值，使用人数数复发mod处理
		console.log(Hash)
		return Hash % 37;
	}
	put(key, value){
		// 根据key得出上面函数返回的随机位置值
		const position = this.getHashTableCode(key)
		this.table[position] = value;
	};
	get(key){
		// 根据key得出上面函数返回的随机位置值
		const position = this.getHashTableCode(key)
		return this.table[position]
	}
	remove(key){
			// 根据key得出上面函数返回的随机位置值
		const position = this.getHashTableCode(key)
			// 由于不能改变数组的长度，不然会影响到其他位置，所以赋值未定义就好
		table[position] = void(0);
	}
}
let hashTable = new HashTable();
hashTable.put('Mark', 'mark@gmail.com');
hashTable.put('Ivy', 'ivy@gmail.com');
hashTable.put('Mary', 'mary@gmail.com');

hashTable.get('Mark');
hashTable.get('Jack');
hashTable.remove('Mark');
hashTable.get('Mark');
```
在上面实际的过程中，会有一个问题，就是可能会有不同的键值但是拥有相同的hash值的情况出现比如Jamie 和 Sue两个字符串获得的hash值就会相同。
```js
getHashTableCode(key){ // 获取随机hash值
		let Hash = 0;
		for(let i = 0; i < key.length; i++) {
			// aharCodeAt 会回传置顶字符串的Unicode编码，所以可以包含中文
			Hash += key.charCodeAt(i);	
		}
		  // 为了取到较小值，使用人数数复发mod处理
		console.log(Hash)
		return Hash % 37;
	}
getHashTableCode('Jamie')
getHashTableCode('Sue')  // 最后结果都是5两个不同的字符串获得了相同的‘随机值’
```

若这种情况不处理，那么后来的值就会覆盖前面的值


```js
getHashTableCode(key){ // 获取随机hash值
		let Hash = 5381;
		for(let i = 0; i < key.length; i++) {
			// aharCodeAt 会回传置顶字符串的Unicode编码，所以可以包含中文
			// ps这里这个33是原文作者瞎写的。。。美其名曰根据经验。。。
			hash = hash * 33 + key.charCodeAt(i);
		}
		  // 为了取到较小值，使用人数数复发mod处理
		console.log(Hash)
		return hash % 1013; // hash 对 1013 取模
	}
```

接下来介绍一下ES6的Map数据结构

Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题，我们扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名。  ---[看的阮一峰的在线书] (http://es6.ruanyifeng.com/#docs/set-map#Map)

size 属性
`size`属性返回Map结构的成员总数
```js
const map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
```
set(key, value)

`set`方法设置键名`key`对应的键值为`value`，然后返回整个 Map 结构。如果`key`已经有值，则键值会被更新，否则就新生成该键。


```javascript
const m = new Map();

m.set('edition', 6)        // 键是字符串
m.set(262, 'standard')     // 键是数值
m.set(undefined, 'nah')    // 键是 undefined
```

`set`方法返回的是当前的`Map`对象，因此可以采用链式写法。
```javascript
let map = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');
```
get(key)

`get`方法读取`key`对应的键值，如果找不到`key`，返回`undefined`。

```javascript
const m = new Map();

const hello = function() {console.log('hello');};
m.set(hello, 'Hello ES6!') // 键是函数

m.get(hello)  // Hello ES6!
```

has(key)

`has`方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。

```javascript
const m = new Map();

m.set('edition', 6);
m.set(262, 'standard');
m.set(undefined, 'nah');

m.has('edition')     // true
m.has('years')       // false
m.has(262)           // true
m.has(undefined)     // true
```
delete(key)

`delete`方法删除某个键，返回`true`。如果删除失败，返回`false`。
```javascript
const m = new Map();
m.set(undefined, 'nah');
m.has(undefined)     // true

m.delete(undefined)
m.has(undefined)       // false
```

clear()
`clear`方法清除所有成员，没有返回值。

```javascript
let map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
map.clear()
map.size // 0
```
遍历方法

Map 结构原生提供三个遍历器生成函数和一个遍历方法。

*   `keys()`：返回键名的遍历器。
*   `values()`：返回键值的遍历器。
*   `entries()`：返回所有成员的遍历器。
*   `forEach()`：遍历 Map 的所有成员。

需要特别注意的是，Map 的遍历顺序就是插入顺序。
