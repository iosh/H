---
title: React中PureComponent使用
date: 2018-04-19 20:44:47
tags: React
---

React中 PureComponent的使用和介绍

<!-- more -->

# 介绍PureComponent

在官方文档中是这样介绍PureComponent的：

> `React.PureComponent` 与 [`React.Component`](https://doc.react-china.org/docs/react-api.html#react.component) 几乎完全相同，但 `React.PureComponent`通过prop和state的浅对比来实现 [`shouldComponentUpate()`](https://doc.react-china.org/docs/react-component.html#shouldcomponentupdate)。
>
> 如果React组件的 `render()` 函数在给定相同的props和state下渲染为相同的结果，在某些场景下你可以使用 `React.PureComponent` 来提升性能。

这段话中，有两个重点：

1. PureComponent 是通过 Props 和 state 浅对比来实现 shouldComponentUpate()，shouldComponentUpate() 生命周期中返回 false 就可以阻止组件重新渲染。
2. 在某些场景下可以提升性能

要理解第一点，首先要明白 JavaScript 中的浅对比和深对比：

首先，这个浅和深只存在于 JavaScript 的 数组和对象， 从字面意思上来说，浅对比只对比前后是否是同一个对象，而深对比则会递归对比对象/数组中的每一个值是否相等。比如一个数组 items ，随后向 items 中 push 一个 字符串，那么浅对比发现前后都是同一个 items，所以不重新渲染，而深对比发现前后里面的值并不相等，从而引发重新渲染。

具体可以看看[这篇文章介绍](http://imweb.io/topic/598973c2c72aa8db35d2e291)

PureComponent 方法改变了 shouldComponentUpate， 并且他会自动检查是否重新渲染，只有检测到 state 或者 props 发生变化时候，才会重新调用 render 方法，而在 Component 中父组件的 render 方法调用后如果不在 shouldComponentUpate 中返回 false 那么子组件也会重新渲染，不管数据是否发生变化。因此，PureComponent 不需要手动写额外的检查

在 React 源码中，PureComponent 是这样进行对比的

```JavaScript
if (this._compositeType === CompositeTypes.PureClass) {
  shouldUpdate = !shallowEqual(prevProps, nextProps) || ! shallowEqual(inst.state, nextState);
}
```

在这其中 shadowEqual 会 浅 检查组件中 props 和 state ，这意味这被嵌套的对象和数组不会进行比较（这就是浅对比）

那么第二点就很好理解了，因为减少了不必要的渲染，所以可以提升性能。

# 使用PureComponent

那么如何使用 PureComponent 呢，因为 PureComponent 如果使用不当，那么他将没什么作用，首先一点是，如果改变组件内部的 props 和 state，那么它将无法发挥作用，下面举个例子。

```JavaScript
import React from 'react';
import ReactDOM from 'react-dom';

function ItemList (props) {
  return (
    <div>
    {props.items.map((item,index) => (
      <div key={index}>{item}</div>
    ))}
    </div>
  )
}


class App extends React.PureComponent { // 如果这里是 React.Component 那么这个组件会正常渲染
  constructor(){
    super()
    this.state={
      items:[1,2,3,4,5,6]
    }
  }
  handleClick = () => {
    // 声明一个变量，储存state
    let { items } = this.state
    // 向items 中 push 一个变量
    items.push('new-item')
    // 调用 this.setState
    this.setState({ items })
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick}>我是按钮啦</button>
        <ItemList items={this.state.items} />
      </div>
    )
  }
}
ReactDOM.render(<App />, document.getElementById('root'));
```



如果这个组件继承 React.Component 那么它没有什么问题，但是如果它继承于 React.PureComponent 那么点击按钮它是不会重新渲染的，因为items前后都是指向同一个对象

需要对代码进行如下修改方可正常重新渲染

```JavaScript
 handleClick = () => {
    this.setState(prevState => ({
      items: prevState.items.concat(['new-item'])
    }))
  }
 
```

 这样，给 items 返回一个全新的对象

那么同时有另外一个问题，如果给 PureComponent 的 state 或者 props 引用了一个新的对象，那么这个组件就会被重新渲染（render）。这意味着如果不想损失它的优点，那么应该避免一下结构

```JavaScript
<ItemList items={this.props.items || []} />
```

这样的结构就导致了ItemList不管接受的是一个Items 还是空数组，都会使组件重新渲染

# 总结

事实上，理解之后使用 PureComponent 是非常简单的，只需要将 Component 换成 PureComponent 即可，这样不但可以平滑过度，提升性能，所以可以在追求性能的时候不妨考虑一下 PureComponent 。

那么说了这么多是不是迫不及待使用 PureComponent 了呢，那么别忘了最重要一点，PureComponent 还会阻止 this.context 改变时候引起的子组件重新渲染，除非在 PureComponent 中声明 contextTypes。（没写过，大概是说，会阻止 redux 引起的重新渲染）

