---
title: JavaScript树结构
date: 2017-12-14 22:15:45
tags: JavaScript
---
```
	研究了半天怎么画图，发现上传hexo没有图，好心酸

		     1                   1
   	   /    \	       /   \
           2      3           2     3
		 / \	/ \         / \   /  \
		4   5   6  7	   4    5     6
			（树）               (图)
```
上面的例子中左边是一棵树，右边的是一个图，因为左边的没有回路，而右边的图存在了1-2-5-3-1这样的回路。
因为树有着*不包含回路*这个特点，所以树被赋予了很多特性。

1.	一棵树中的任意两个节点有且仅有唯一一条路径连通。
2.	树如果有n个基点，name他一定恰好有n-1条边。
3.	在一棵树中加一条边将会构成一个回路。
在JavaScript有很多树形结构，比如DOM树等。
常见用途有晋级图、家谱、以及系统的文件路径。

首先*树*指有任意两个节点间有且只有一条路径的无向图，或者说只要是没有回路的无向图就是树。

为了确定一棵树的形态，在树种可以指定一个特殊的节点---*根*，在对一棵树进行讨论时，将书中的每个点成为结点也可以称为节点。有一个根的书叫做树根，根又叫做根节点有时也称作祖先。

###  二叉树

二叉树是一种特殊的树，二叉树的特点是每个节点最多有两个子节点，左边叫做左节点，右边成为右节点，或者说每个几点最多有两颗子树，更加严格的递归定义：*二叉树要么为空，要么由根节点、左子树、右子树组成，而左、右子树，分别是一颗二叉树*

画图好难。。我不画了

二叉树还有两种特殊的二叉树，分别叫做*满二叉树*和*完全二叉树*，如果二叉树中每个内部节点都有两个子节点，这样的二叉树叫做满二叉树，或者说满二叉树所有的叶节点都有相同的深度。

比如这样

```
      1
    /   \
   2     3
  / \   / \
 4   5 6   7

这个简单的图会了

```

如果一颗二叉树除了最右边位置上有一个或者几个叶节点缺少外，其他都是丰满的，name这样的二叉树就是完全的二叉树，严格的定义是：*若设二叉树的高度为h,除底h层之外，其他各层（1~h-1）的节点都达到最大个数，第h层从右向左连续缺若干节点，则这个二叉树就是完全二叉树，也就是说如果一个节点有右子节点，那么他一定也有左子节点。*如下图都是完全二叉树，也可以将满二叉树理解成一种特殊或者极其完美的完全二叉树。

```
      1               1             1
    /   \           /   \         /   \
   2     3         2     3       2     4
  /               / \           /
 4               4   5         3

```

重点: *满二叉树一定是完全二叉树，但是完全二叉树不一定是满二叉树*


###  二叉树的性质

一：在二叉树的第n层上至多有2^(n-1)个结点(n>=1)
二：深度为n的二叉树至多有2^n-1个结点(n>=1)



###  二叉链表

二叉树的储存按照国际惯例来说，一般采用链式储存结构，二叉树每个节点最多有两个子节点。

###  二叉树的遍历

二叉树的遍历是指从根节点出发，按照某种顺序依次访问二叉树中所有节点，使得每个节点被访问依次且仅被访问依次。

###  实现查找二叉树
二叉搜索树只允许你在左侧储存比父节点小的值，右侧节点储存比父节点大或者相等的值。
```js
function Node(data,left,right){ // 声明一个构造函数
	this.data = data;   
 	this.left = left;   // 左树
	this.right = right; // 右树
	this.show = function (){ // 显示data存的数据
		return this.data
	}
}
```
定义一个二叉树

```js
function BST() {
	this.root = null; // 初始节点为空根节点为空
	this.insert = function(data) { // 插入数据方法
		let n = new Node(data,null,null) // 实例化Node并传入初始化数据，左右树为赋值为空
		if(!this.root){
			this.root = n;  // 判断root是否(初始)为空如果为空则赋值为Node实例	
		} else {
			let current = this.root;
			let parent;
			while(true) { // 一直循环执行代码块直到满足条件break跳出循环
				parent = current;
				if(data < current.show()){// 判断要插入的数据是否大于根节点的值如果大于等于则在右树小于则在右树
					current = current.left; // 小于根节点在左树，current储存左树节点
					if(!current){ // 如果左节点为空则储存,不为空则继续while判断
						parent.left = n;
						break;		
					}
				} else {
					current = current.right;
					if(!current) {
						parent.right = n;
						break;
					}
				}
			}
		}
	}
}
谷歌控制台试一下
var a = new BST()
a.insert(1)
a.insert(2)
a.insert(4)
a.insert(3)
a.insert(6)
a.insert(7)
a.insert(8)
a
然后看一下结构。是否都是在右树上（因为根节点是1后面的数都比1大所以都在右树）
```
获取我们生成的二叉树最小值、最大值

```js
function BST() {
	this.root = null; // 初始节点为空根节点为空
	this.insert = function(data) { // 插入数据方法
		let n = new Node(data,null,null) // 实例化Node并传入初始化数据，左右树为赋值为空
		if(!this.root){
			this.root = n;  // 判断root是否(初始)为空如果为空则赋值为Node实例	
		} else {
			let current = this.root;
			let parent;
			while(true) { // 一直循环执行代码块直到满足条件break跳出循环
				parent = current;
				if(data < current.show()){// 判断要插入的数据是否大于根节点的值如果大于等于则在右树小于则在右树
					current = current.left; // 小于根节点在左树，current储存左树节点
					if(!current){ // 如果左节点为空则储存,不为空则继续while判断
						parent.left = n;
						break;		
					}
				} else {
					current = current.right;
					if(!current) {
						parent.right = n;
						break;
					};
				};
			};
		};
	};
	// 定义一个方法查找最小值
	this.getMin = function() {
		if(!this.root) return '二叉树是空的';
		let current = this.root; // 变量暂存总数
		while(current.left){ // 小的都在左树循环判断有没有左节点就好了
			current = current.left // 如果有左树则存左树
		}
		return current.show()
	}；
	// 定义一个方法查找最大值
	this.getMax = function() {
    	if (!this.root) return null;
    	let current = this.root;
    	while (current.right) {
        	current = current.right;
    	}
    	return current.show();
	}
}
试一下
var a = new BST()
a.insert(7) // 根
a.insert(6) // 左树
a.insert(8)  // 右树
a.insert(90) // 右树
a.insert(3)  // 左树
a.getMin() // 3
a.getMax() // 90
```

###  中序遍历

规则是先遍历左节点，然后到根节点，最后到右节点
![](https://upload.wikimedia.org/wikipedia/commons/thumb/7/77/Sorted_binary_tree_inorder.svg/440px-Sorted_binary_tree_inorder.svg.png)

```js
BST.prototype.inOrder = function(node) {
    if (node) {
        inOrder(node.left);
        console.log(node.show());
        inOrder(node.right);
    }
}
```
### 先序遍历

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d4/Sorted_binary_tree_preorder.svg/440px-Sorted_binary_tree_preorder.svg.png)

```js
BST.prototype.preOrder = function(node) {
    if (node) {
        console.log(node.show());
        preOrder(node.left);
        preOrder(node.right);
    }
}
```

### 后序遍历

![](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9d/Sorted_binary_tree_postorder.svg/440px-Sorted_binary_tree_postorder.svg.png?1445511399697)

```js
BST.prototype.postOrder = function(node) {
    if (node) {
        postOrder(node.left);
        postOrder(node.right);
        console.log(node.show());
    }

}
```

### 查找

查找其实相对简单，毕竟查找数值与当前结点，小则循环到左节点，大则循环到右节点，等于则返回，查找不到返回 null。
```js
BST.prototype.find = function(data) {
    let current = this.root;
    while (current) {
        if (Object.is(data, current.show())) {
            return current;
        } else if (data < current.show()) {
            current = current.left;
        } else {
            current = current.right;
        }
    }
    return null;
}
```