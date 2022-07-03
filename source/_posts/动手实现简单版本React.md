---
title: 动手实现简单版本React
date: 2022-07-03 12:46:38
tags: React
---

实现要给简单的 React

<!-- more -->

通过动手实现一个简单的 React 来进行深入学习

# 准备

为了简单起见这里使用 create-react-app 创建一个 js 项目， 之后将代码写在一个 index.js 文件中。

```javascript
import React from "react";

import ReactDOM from "react-dom";

const App = <div className="test">hi</div>;

const container = document.getElementById("root");

ReactDOM.render(App, container);
```

上面就是 React 最简单的例子， 有一个名为 App 的 React Element， 有一个 id 为 root 的 DOM 节点， 最终由 react-dom 的 render 方法渲染出来，这里先不考虑新的渲染 API。

```javascript
// babel 会将

const App = <div className="test">hi</div>;

// 编译为

const App = /*#__PURE__*/ React.createElement(
  "div",
  {
    className: "test",
  },
  "hi"
);
```

# 第一版

## createElement

babel 会将 jsx 转化为实际的函数，所以首先需要实现一个 createElement 函数来做同样的方法， createElement 由三个参数， 第一个参数是组件， 或者 HTML 的 tag 名称， 第二个参数是 props， 第三个参数是 children

```javascript
function createElement(type, props, children) {
  return {
    type,
    props: { ...props, children },
  };
}
```

先实现一个简单的 createElement 函数

之后尝试渲染一下， 首先在项目根目录新建一个`.env`文件

并输入

```text
DISABLE_NEW_JSX_TRANSFORM = true
```

然后重新启动项目， 这一个环境变量会阻止 babel 自动引入 react， 并允许使用注释来决定 jsx 渲染

```javascript
import ReactDOM from "react-dom";

function createElement(type, props, children) {
  return {
    type,
    props: { ...props, children },
  };
}

const toyReact = {
  createElement,
};

/** @jsx toyReact.createElement */
const App = <div className="test">hi</div>;

const container = document.getElementById("root");

ReactDOM.render(App, container);
```

启动项目，不出意外的话会得到一个报错， 因为 React 内部会检查返回的 jsx 对象， 并判断类型， 如果是 object 类型会判断是否存在 $$typeof 属性， 如果不存在会报错。

那么需要我们自己实现一个 render 函数

## render

已经知道 createElement 只是返回一个对象， 那么我们的 render 就需要根据这个对象来生成 DOM，并显示在页面之上。

```javascript
function createElement(type, props, children) {
  return {
    type,
    props: { ...props, children },
  };
}

function render(jsxElement, container) {
  const { type, props } = jsxElement;

  const node = document.createElement(type);
  if (props.className) {
    node.classList.add(props.className);
  }

  const text = document.createTextNode(props.children);

  node.appendChild(text);
  container.appendChild(node);
}

const toyReact = {
  createElement,
  render,
};

/** @jsx toyReact.createElement */
const App = <div className="test">hi</div>;

const container = document.getElementById("root");

toyReact.render(App, container);
```

这样就可以简单的在页面上显示一段文字。 但是仅仅显示一行文字是不够的，在实际中 jsx 都是多层嵌套的， 例如

```javascript
const App = (
  <div>
    <span> hello </span> <span>world</span>
  </div>
);
```

# 第二版

## createElement

```javascript
const App = (
  <div>
    <span>hello</span>
    <span>world</span>
  </div>
);

// 经过转化会转变为

const App = /*#__PURE__*/ toyReact.createElement(
  "div",
  null,
  /*#__PURE__*/ toyReact.createElement("span", null, "hello"),
  /*#__PURE__*/ toyReact.createElement("span", null, "world")
);
```

这里可以修改一下 createElement

```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: { ...props, children },
  };
}
```

通过使用 ... 剩余参数 , 将所有参数收集成为一个数组。这样无论有多少子元素，都可以会在一个数组里。为了方便 render 函数处理， 这里需要特殊识别一下文本元素。

```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map((child) =>
        typeof child === "string" ? createTextElement(child) : child
      ),
    },
  };
}
```

这里判断如果 child 类型是 字符串 那么我们将调用 createTextElement 创建要给对象

```javascript
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}
```

通过这个对象，我们就可以通过同一种方法来识别元素每个元素都是将是一个对象

```JavaScript
const App = (
  <div>
    <span>hello </span>
    <span>world</span>
  </div>
);
// 将被转化为


{
  "type": "div",
  "props": {
    "children": [
      {
        "type": "span",
        "props": {
          "children": [
            {
              "type": "TEXT_ELEMENT",
              "props": { "nodeValue": "hello ", "children": [] }
            }
          ]
        }
      },
      {
        "type": "span",
        "props": {
          "children": [
            {
              "type": "TEXT_ELEMENT",
              "props": { "nodeValue": "world", "children": [] }
            }
          ]
        }
      }
    ]
  }
}

```

接下来我们的 render 函数只需要遍历这个结构即可

## render

```javascript
function render(jsxElement, container) {
  const { type, props } = jsxElement;

  const node =
    type !== "TEXT_ELEMENT"
      ? document.createElement(type)
      : document.createTextNode(props.nodeValue);

  props.children.forEach((child) => render(child, node));

  if (props.className) {
    node.classList.add(props.className);
  }

  container.appendChild(node);
}
```

render 函数还要支持更多的 dom 属性

```javascript
const App = (
  <div>
    <span
      className="hello"
      style={{ color: "red" }}
      onClick={() => console.log("hello")}
    >
      hello
    </span>
    <span>world</span>
  </div>
);
```

支持 className style 和 一些事件

```javascript
function render(jsxElement, container) {
  const {
    type,
    props: { children, ...rest },
  } = jsxElement;

  const node =
    type !== "TEXT_ELEMENT"
      ? document.createElement(type)
      : document.createTextNode(rest.nodeValue);

  // 在这里处理 props 的属性
  Object.keys(rest).forEach((name) => {
    if (name === "className") {
      node.class = rest[name];
    } else if (name === "style") {
      Object.keys(rest.style).forEach(
        (styleKey) => (node.style[styleKey] = rest.style[styleKey])
      );
    } else {
      node[name.toLowerCase()] = rest[name];
    }
  });

  children.forEach((child) => render(child, node));

  container.appendChild(node);
}
```

至此我们已经可以将 jsx 渲染为实际 dom 的两个函数, 可以看到这里 render 是使用了递归进行操作的， 递归函数开始之后就不能停止，在完全渲染完成之前，会一直占用 JavaScript 线程，如果节点足够多，用户就会感觉到 _卡顿_ 因为 js 线程被用来递归， 从而不会相应用户的任何操作。

下面是全部代码

```javascript
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}

function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map((child) =>
        typeof child === "string" ? createTextElement(child) : child
      ),
    },
  };
}

function render(jsxElement, container) {
  const {
    type,
    props: { children, ...rest },
  } = jsxElement;

  const node =
    type !== "TEXT_ELEMENT"
      ? document.createElement(type)
      : document.createTextNode(rest.nodeValue);

  Object.keys(rest).forEach((name) => {
    if (name === "className") {
      node.class = rest[name];
    } else if (name === "style") {
      Object.keys(rest.style).forEach(
        (styleKey) => (node.style[styleKey] = rest.style[styleKey])
      );
    } else {
      node[name.toLowerCase()] = rest[name];
    }
  });

  children.forEach((child) => render(child, node));

  container.appendChild(node);
}

const toyReact = {
  createElement,
  render,
};

/** @jsx toyReact.createElement */
const App = (
  <div>
    <span
      className="hello"
      style={{ color: "red" }}
      onClick={() => console.log("hello")}
    >
      hello
    </span>
    <span>world</span>
  </div>
);

console.log(JSON.stringify(App));
const container = document.getElementById("root");

toyReact.render(App, container);
```

# 第三版

为了防止递归长时间占用线程，需要将整个渲染工作拆分成若干小的工作，每次浏览器空闲的时候都进行我们的工作。

React 解决这个问题的办法是创建了一种 fiber 数据结构，这种结构可以方便的链接父节点子节点兄弟节点，这样每个工作单元只处理个节点，然后下次在处理子节点， 如果没有子节点那么看看是否有兄弟节点，有的话就处理兄弟节点。

首先将创建 DOM 的部分独立出来, 将原来 render 函数内的部分转移到 createDOM 函数内

```javascript
function createDOM(fiber) {
  const {
    type,
    props: { children, ...rest },
  } = fiber;

  const node =
    type !== "TEXT_ELEMENT"
      ? document.createElement(type)
      : document.createTextNode(rest.nodeValue);

  Object.keys(rest).forEach((name) => {
    if (name === "className") {
      node.class = rest[name];
    } else if (name === "style") {
      Object.keys(rest.style).forEach(
        (styleKey) => (node.style[styleKey] = rest.style[styleKey])
      );
    } else {
      node[name.toLowerCase()] = rest[name];
    }
  });

  return node;
}
```

这个函数将根据 fiber 节点来创建 dom。

```javascript
// 这个变量用来储存将要处理的节点
let nextUnitOfWork = null;
// 这个用来储存原始节点
let workInProcessRoot = null;

function render(jsxElement, container) {
  workInProcessRoot = {
    dom: container,
    props: {
      children: [jsxElement],
    },
  };

  nextUnitOfWork = workInProcessRoot;
  requestIdleCallback(workLoop);
}
```

```javascript
function workLoop(deadline) {
  let shouldYield = false;

  while (nextUnitOfWork && !shouldYield) {
    // 获取下一个需要渲染的元素
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    // 判断剩余时间是否大于1
    shouldYield = deadline.timeRemaining() < 1;
  }

  // 如果没有下一个节点，并且存在原始节点那么就是要到将dom添加到浏览器里面的阶段了
  if (!nextUnitOfWork && workInProcessRoot) {
    commitRoot();
  }

  requestIdleCallback(workLoop);
}
```

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    // 判断 fiber 对象上是否有dom节点
    fiber.dom = createDOM(fiber);
  }

  const { children } = fiber.props;

  // 接下来处理子节点
  let prevSibling = null;
  for (let index = 0; index < children.length; index++) {
    const { type, props } = children[index];
    const newFiber = {
      type,
      props,
      parent: fiber,
      dom: null,
    };
    if (index === 0) {
      // 指向子节点
      fiber.child = newFiber;
    } else {
      // 设置兄弟节点的指针
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
  }
  // 处理完成子节点之后，返回下一个待处理的节点
  if (fiber.child) return fiber.child;

  // 如果没有子节点那么返回兄弟节点 （如果兄弟节点也没有说明结束了
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}
```

```javascript
function commitWork(fiber) {
  if (!fiber) return;
  const { dom } = fiber.parent;
  dom.appendChild(fiber.dom);
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

// 使用初始节点（root）节点来将fiber 添加到dom中

function commitRoot() {
  commitWork(workInProcessRoot.child);
  workInProcessRoot = null;
}
```

这里已经可以通过 fiber 对象来渲染节点，并且不适用递归。 下面是完整代码

```javascript
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}

function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map((child) =>
        typeof child === "string" ? createTextElement(child) : child
      ),
    },
  };
}

function createDOM(fiber) {
  const {
    type,
    props: { children, ...rest },
  } = fiber;

  const node =
    type !== "TEXT_ELEMENT"
      ? document.createElement(type)
      : document.createTextNode(rest.nodeValue);

  Object.keys(rest).forEach((name) => {
    if (name === "className") {
      node.class = rest[name];
    } else if (name === "style") {
      Object.keys(rest.style).forEach(
        (styleKey) => (node.style[styleKey] = rest.style[styleKey])
      );
    } else {
      node[name.toLowerCase()] = rest[name];
    }
  });

  return node;
}

function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }

  const { children } = fiber.props;

  let prevSibling = null;
  for (let index = 0; index < children.length; index++) {
    const { type, props } = children[index];
    const newFiber = {
      type,
      props,
      parent: fiber,
      dom: null,
    };
    if (index === 0) {
      fiber.child = newFiber;
    } else {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
  }
  if (fiber.child) return fiber.child;

  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}

let nextUnitOfWork = null;
let workInProcessRoot = null;

function commitWork(fiber) {
  if (!fiber) return;

  const { dom } = fiber.parent;
  dom.appendChild(fiber.dom);
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitRoot() {
  commitWork(workInProcessRoot.child);
  workInProcessRoot = null;
}

function workLoop(deadline) {
  let shouldYield = false;

  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }

  if (!nextUnitOfWork && workInProcessRoot) {
    commitRoot();
  }

  requestIdleCallback(workLoop);
}

function render(jsxElement, container) {
  workInProcessRoot = {
    dom: container,
    props: {
      children: [jsxElement],
    },
  };

  nextUnitOfWork = workInProcessRoot;
  requestIdleCallback(workLoop);
}

const toyReact = {
  createElement,
  render,
};

/** @jsx toyReact.createElement */
const App = (
  <div>
    <span
      className="hello"
      style={{ color: "red" }}
      onClick={() => console.log("hello")}
    >
      hello
    </span>
    <span>world</span>
  </div>
);

const container = document.getElementById("root");

toyReact.render(App, container);
```

