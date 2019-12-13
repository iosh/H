---
title: JavaScript栈、队列和链表
date: 2017-12-13 20:34:00
tags: JavaScript
---

### 队列

队列是一种*先进先出* 的数据结构，类似于排队点餐，排在第一个的就可以第一个点餐，而后面的只能按照队列顺序等待执行。

<!-- more -->

```js
function Queue() {
  let items = [];
  this.enqueue = function(element) {
    // enqueue方法向队列添加一个或者多个元素
    items.push(element);
  };
  this.dequeue = function() {
    // dequeue 方法移除队列的第一个元素并返回移除的值
    return items.shift();
  };
  this.front = function() {
    // front 方法返回列队第一个元素，队列不做更改
    return items[0];
  };

  this.clear = function() {
    // clear 方法移除队列所有的元素
    items = [];
  };

  this.isEmpty = function() {
    //如果队列内没有任何元素就返回true，否则返回false
    return items.length == 0;
  };

  this.size = function() {
    // size方法返回队列里的元素个数
    return items.length;
  };
  this.print = function() {
    // print方法打印
    console.log(items);
  };
}
var queue = new Queue();
queue.enqueue(2); // 添加一个元素
queue.print(); // [2] 打印列队
queue.size(); // 1 队列长度
```

简单的实现了一个列队构造函数

简单的运用一下

> 优先队列

    队列的添加和移除都是基于优先级，实现一个优先队列，有两种选项，设置优先级，然后再正确的位置添加元素，或者用入队操作添加元素，然后按照优先级移除。

```js
function PriorityQueue() {
  const items = []; // 声明一个数组
  // 声明一个构造函数，接受两个参数，普通和优先
  function QueueElement(element, priority) {
    this.element = element;
    this.priority = priority;
  }
  // 声明一个构造函数方法接受两个参数同上
  this.enqueue = function(element, priority) {
    // 声明一个变量实例化QueueElement
    const queueElement = new QueueElement(element, priority);
    // 判断items是否为空，如果是空直接插入无需排队
    if (items.length === 0) {
      items.push(queueElement);
    } else {
      // 如果items不为空则循环判断
      let added = false;
      // 循环items的每一项
      for (var i = 0; i < items.length; i++) {
        // 判断当前的优先是否大于已有列队的优先数字
        if (queueElement.priority < items[i].priority) {
          // 如果找到当前优先大于列队优先则在这里插入
          items.splice(i, 0, queueElement);
          added = true;
          break;
        }
      }
    }
    // 如果优先度不如列队里面的则加到最后一个
    if (!added) {
      items.push(queueElement);
    }
  };
  // 出队方法
  this.dequeue = function() {
    return items.shift();
  };
  // 查询第一个
  this.front = function() {
    return items[0];
  };
  // 判断列队是否为空
  this.isEmpty = function() {
    return items.length == 0;
  };
  // 判断长度
  this.size = function() {
    return items.length;
  };
  // 打印
  this.print = function() {
    for (var i = 0; i < items.length; i++) {
      console.log(items[i].element + " - " + items[i].priority);
    }
  };
}
```

> 循环列队
> 击鼓传花
> 击鼓传花游戏，在这个游戏中，孩子们围成一个圆圈，把花尽快的传递给旁边的人。某一时刻传花停止，这个时候花落在谁手里，谁就退出圆圈结束游戏。重复这个过程，直到只剩下一个孩子

```js
// 声明一个函数 接受两个参数名称列表和总数
function hotPotato(namelist, num) {
  // 声明一个变量实例化Queue
  var queue = new Queue();
  // 循环玩家名称，然后添加到列队里面
  for (var i = 0; i < namelist.length; i++) {
    queue.enqueue(namelist[i]);
  }
  var eliminated = "";
  // 判断列队是否大于1即玩家是否大于两位
  while (queue.size() > 1) {
    for (var i = 0; i < num; i++) {
      // 删除第一个
      queue.enqueue(queue.dequeue());
    }
    eliminated = queue.dequeue();
    console.log(eliminated + "在游戏中淘汰了。");
  }
  return queue.dequeue();
}
var names = ["a", "b", "c", "d", "e"];
var winner = hotPotato(names, 7);
console.log("胜利者" + winner);
//c在游戏中淘汰了。
//b在游戏中淘汰了。
//e在游戏中淘汰了。
//d在游戏中淘汰了。
//胜利者a
```

### 栈

栈，是一种先进后出的有序集合，新添加或待删除的元素都保存在栈的末位，称作栈顶，另一端成为栈底，新元素都靠近栈顶，就元素都接近栈底。

```
function Stack() {

  /**
   * 用数组来模拟栈
   */
  var items = [];

  /**
   * 将元素送入栈，放置于数组的最后一位
   */
  this.push = function(element) {
    items.push(element);
  };

  /**
   * 弹出栈顶元素
   */
  this.pop = function() {
    return items.pop();
  };

  /**
   * 查看栈顶元素
   */
  this.peek = function() {
    return items[items.length - 1];
  }

  /**
   * 确定栈是否为空
   * @return {Boolean} 若栈为空则返回true,不为空则返回false
   */
  this.isAmpty = function() {
    return items.length === 0
  };

  /**
   * 清空栈中所有内容
   */
  this.clear = function() {
    items = [];
  };

  /**
   * 返回栈的长度
   * @return {Number} 栈的长度
   */
  this.size = function() {
    return items.length;
  };

  /**
   * 以字符串显示栈中所有内容
   */
  this.print = function() {
    console.log(items.toString());
  };
}
```

十进制转任何进制代码

```
  /**
   * decNumber 要转换的十进制数字
   * base      目标进制基数
   */
  function baseConverter(decNumber, base) {
    var remStack = new Stack(),
        rem,
        baseString = '',
        digits = '0123456789ABCDEF';
    white (decNumber > 0) {
        rem = Math.floor(decNumber % base);
        remStack.push(rem);
        decNumber = Math.floor(decNumber / base);
    }

    white(!remStack.isEmpty()) {
        baseString += digits[remStack.pop()];
    }

    return baseString;
  }
```

### 链表

链表存储有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的。每个元素由一个存储元素本事的节点和一个指向下一个元素的引用组成。相对于传统的数组，链表的一个好处在于，添加或者删除元素的时候不需要移动其他元素。然而，链表需要使用指针，因此实现链表时需要额外注意。

> 数组和链表的一个不同在于数组可以直接访问任何位置的元素，而想要访问链表中的一个元素，需要从起点开始迭代列表。

> 单向链表

```js
function LinkedList() {
  var Node = function(element) {
    this.element = element;
    this.next = null;
  };
  var length = 0; //链表长度
  var head = null; //第一个节点
  this.append = function(element) {
    var node = new Node(element),
      current;
    if (head === null) {
      //列表为空
      head = node;
    } else {
      //列表不为空
      current = head; //现在只知道第一项head
      while (current.next) {
        //找到列表的最后一项
        current = current.next;
      }
      //建立链接
      current.next = node;
    }
    length++; //更新列表长度
  };
  this.insert = function(position, element) {
    //检查越界值
    if (position >= 0 && position <= length) {
      var node = new Node(element),
        current = head,
        previous,
        index = 0;
      if (position === 0) {
        //在第一个位置添加
        node.next = current;
        head = node;
      } else {
        //在中间或者尾部添加
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        node.next = current; //先连上添加的节点
        previous.next = node; //再断开之前的连接
      }
      length++;
      return true;
    } else {
      return false;
    }
  };
  this.removeAt = function(position) {
    if (position > -1 && position < length) {
      var current = head,
        previous,
        index = 0; //用来迭代列表，直到到达目标位置
      if (position === 0) {
        //移除第一项
        head = current.next;
      } else {
        //移除中间或者尾部最后一项
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        //连接前一项和后一项，跳过当前的项，相当于移除了当前项
        previous.next = current.next;
      }
      length--;
      return current.element;
    } else {
      return null;
    }
  };
  this.remove = function(element) {
    var index = this.indexOf(element);
    return this.removeAt(index);
  };
  this.indexOf = function(element) {
    var current = head,
      index = 0;
    while (current) {
      if (element === current.element) {
        return index;
      }
      index++; //记录位置
      current = current.next;
    }
    return -1;
  };
  this.isEmpty = function() {
    return length === 0;
  };
  this.size = function() {
    return length;
  };
  this.getHead = function() {
    return head;
  };
  this.toString = function() {
    var current = head,
      string = "";
    while (current) {
      string += current.element; //拼接
      current = current.next;
    }
    return string;
  };
  this.print = function() {
    console.log(this.toString());
  };
}
```

> 双向链表

双向链表和单向链表的区别在于，在单向链表中，一个节点只有链向下一个节点的链接。而在双向链表中，链接是双向的：一个链向下一个元素，另一个链向前一个元素

```js
function DoublyLinkedList() {
  var Node = function(element) {
    this.element = element;
    this.next = null;
    this.prev = null; //新添加的
  };
  var length = 0;
  var head = null;
  var tail = null; //新添加的
  this.append = function(element) {
    var node = new Node(element),
      current;
    if (head === null) {
      //列表为空
      head = node;
      tail = node;
    } else {
      tail.next = node;
      node.prev = tail;
      tail = node;
    }
    length++;
  };
  this.insert = function(position, element) {
    if (position >= 0 && position <= length) {
      var node = new Node(element),
        current = head,
        previous,
        index = 0;
      if (position === 0) {
        //在第一个位置
        if (!head) {
          //列表为空
          head = node;
          tail = node;
        } else {
          //列表不为空
          node.next = current;
          current.prev = node;
          head = node;
        }
      } else if (position === length) {
        //最后一项
        current = tail;
        current.next = node;
        node.prev = current;
        tail = node;
      } else {
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        node.next = current;
        previous.next = node; //把node节点连接进去前一个节点和后一个节点

        current.prev = node; //断掉之前previous和current的连接
        node.prev = previous; //prev同样需要连接
      }
      length++;
      return true;
    } else {
      return false;
    }
  };
  this.removeAt = function(position) {
    if (position > -1 && position < length) {
      var current = head,
        previous,
        index = 0;
      if (position === 0) {
        //移除第一项
        head = current.next;
        if (length === 1) {
          // 列表只有一项
          tail = null;
        } else {
          head.prev = null;
        }
      } else if (position === length - 1) {
        移除最后一项;
        current = tail; // {4}
        tail = current.prev;
        tail.next = null;
      } else {
        while (index++ < position) {
          previous = current;
          current = current.next;
        }
        previous.next = current.next; // 链接前一项和后一项，跳过当前项
        current.next.prev = previous; //修复prev
      }
      length--;
      return current.element;
    } else {
      return null;
    }
  };
  this.remove = function(element) {
    var index = this.indexOf(element);
    return this.removeAt(index);
  };
  this.indexOf = function(element) {
    var current = head,
      index = -1;
    //检查第一项
    if (element == current.element) {
      return 0;
    }
    index++;
    //检查中间项
    while (current.next) {
      if (element == current.element) {
        return index;
      }
      current = current.next;
      index++;
    }
    //检查最后一项
    if (element == current.element) {
      return index;
    }
    return -1;
  };
  this.isEmpty = function() {
    return length === 0;
  };
  this.size = function() {
    return length;
  };
  this.toString = function() {
    var current = head,
      s = current ? current.element : "";
    while (current && current.next) {
      current = current.next;
      s += ", " + current.element;
    }
    return s;
  };
  this.inverseToString = function() {
    var current = tail,
      s = current ? current.element : "";
    while (current && current.prev) {
      current = current.prev;
      s += ", " + current.element;
    }
    return s;
  };
  this.print = function() {
    console.log(this.toString());
  };
  this.printInverse = function() {
    console.log(this.inverseToString());
  };
  this.getHead = function() {
    return head;
  };
  this.getTail = function() {
    return tail;
  };
}
```

> 循环链表

循环链表可以像单向链表那样只有单向引用，也可以像双向链表那样有双向引用。循环链表和其他链表的区别在于最后一个元素指向下一个元素的引用不是 null，而是指向第一个元素

```
function CircularLinkedList() {
    var Node = function(element){
        this.element = element;
        this.next = null;
    };
    var length = 0;
    var head = null;
    this.append = function(element){
        var node = new Node(element),
            current;
        if (head === null){ //列表为空
            head = node;
        } else {
            current = head;
            while(current.next !== head){ //最后一个元素将是head，而不是null
                current = current.next;
            }
            current.next = node; //建立连接
        }
        node.next = head; //首尾相连起来变成一个环列表
        length++;
    };
    this.insert = function(position, element){
        if (position >= 0 && position <= length){
            var node = new Node(element),
                current = head,
                previous,
                index = 0;
            if (position === 0){ //在第一项
                node.next = current;
                while(current.next !== head){
                    current = current.next;
                }
                head = node;
                current.next = head;
            } else {
                while (index++ < position){
                    previous = current;
                    current = current.next;
                }
                node.next = current;
                previous.next = node;
                if (node.next === null){ //在最后一个元素更新
                    node.next = head;
                }
            }
            length++;
            return true;
        } else {
            return false;
        }
    };
    this.removeAt = function(position){
        if (position > -1 && position < length){
            var current = head,
                previous,
                index = 0;
            if (position === 0){
                while(current.next !== head){
                    current = current.next;
                }
                head = head.next;
                current.next = head; //更新最后一项
            } else {
                while (index++ < position){
                    previous = current;
                    current = current.next;
                }
                previous.next = current.next;
            }
            length--;
            return current.element;
        } else {
            return null;
        }
    };
    this.remove = function(element){
        var index = this.indexOf(element);
        return this.removeAt(index);
    };
    this.indexOf = function(element){
        var current = head,
            index = -1;
        if (element == current.element){ //检查第一项
            return 0;
        }
        index++;
        while(current.next !== head){ //检查列表中间
            if (element == current.element){
                return index;
            }
            current = current.next;
            index++;
        }
        if (element == current.element){ //检查最后一项
            return index;
        }
        return -1;
    };
    this.isEmpty = function() {
        return length === 0;
    };
    this.size = function() {
        return length;
    };
    this.getHead = function(){
        return head;
    };
    this.toString = function(){
        var current = head,
            s = current.element;
        while(current.next !== head){
            current = current.next;
            s += ', ' + current.element;
        }
        return s.toString();
    };
    this.print = function(){
        console.log(this.toString());
    };
}
```
