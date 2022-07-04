---
title: 动手实现简单版本React
date: 2022-07-03 12:46:38
tags: React
---

实现一个简单的 React

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

这里已经可以通过 fiber 对象来渲染节点，并且使用递归。 下面是完整代码

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

# 第四版

目前为止已经可以正常显示， 现在处理新增或者删除。

```javascript
let nextUnitOfWork = null;
let workInProcessRoot = null;
let currentRoot = null;

function commitRoot() {
  // ....
  workInProcessRoot = null;
}

function render(jsxElement, container) {
  workInProcessRoot = {
    dom: container,
    props: {
      children: [jsxElement],
    },
    alternate: currentRoot,
  };

  //...
}
```

这里新增一个变量用来储存上一次渲染的 fiber 对象， 并且为为 fiber 对象添加一个 alternate 属性，这个属性是上一次渲染 dom 的 fiber 对象

新增一个 reconcileChildren 方法来处理旧的 fiber 和新的元素

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }

  const { children } = fiber.props;
  reconcileChildren(fiber, children);

  if (fiber.child) return fiber.child;

  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}
```

然后实现 reconcileChildren 函数

```javascript
function reconcileChildren(fiber, elements) {
  let oldFiber = fiber.alternate && fiber.alternate.child;

  let prevSibling = null;
  let index = 0;
  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;
    const sameType = oldFiber && element && oldFiber.type === element.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: fiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      };
    }

    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: fiber,
        alternate: null,
        effectTag: "PLACEMENT",
      };
    }

    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      fiber.child = newFiber;
    } else {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}
```

可以看出我们通过对比前后 props 来给 fiber 新增了一个 effectTag 属性，来标记这个 element 应该要做什么。 然后使用 alternate 来储存当前渲染的 fiber 对象

```javascript
function commitWork(fiber) {
  if (!fiber) return;

  const { dom } = fiber.parent;

  if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
    dom.appendChild(fiber.dom);
  } else if (fiber.effectTag === "DELETION") {
    dom.removeChild(fiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
    updateDom(fiber.dom, fiber.alternate.props, fiber.props);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

这样我们就可以根据不同的状态来做不同的事情。

接下来实现 updateDOM 函数

```javascript
// 帮助函数
function compareProps(prevProps, nextProps, callback) {
  // 首先遍历旧的props
  Object.keys(prevProps).forEach((k) =>
    callback(k, prevProps[k], nextProps[k])
  );

  // 然后再遍历新的 props， 并且旧的 props里面没有的项单独拿来处理
  Object.keys(nextProps).forEach((k) => {
    if (!prevProps[k]) {
      callback(k, undefined, nextProps[k]);
    }
  });
}
```

```javascript
function updateDOM(dom, prevProps, nextProps) {
  compareProps(prevProps, nextProps, (k, prevPropsValue, nextPropsValue) => {
    if (prevPropsValue === nextPropsValue || k === "children") {
      // pass
    } else if (k[0] === "o" && k[1] === "n") {
      const eventName = k.substring(2).toLowerCase();
      if (prevPropsValue) dom.removeEventListener(eventName, prevPropsValue);
      dom.addEventListener(eventName, nextPropsValue);
    } else if (k === "style") {
      compareProps(
        prevPropsValue,
        nextPropsValue,
        (styleKay, prevStyle, nextStyle) => {
          if (prevStyle !== nextStyle) {
            dom[k][styleKay] = nextStyle || "";
          }
        }
      );
    } else if (k in dom) {
      dom[k] = nextPropsValue;
    } else if (prevProps === null || nextProps === false) {
      dom.removeAttribute(k);
    } else {
      dom.setAttribute(k, nextPropsValue);
    }
  });
}
```

完整代码如下

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

function compareProps(prevProps, nextProps, callback) {
  prevProps = prevProps || {};
  nextProps = nextProps || {};
  Object.keys(prevProps).forEach((k) =>
    callback(k, prevProps[k], nextProps[k])
  );

  Object.keys(nextProps).forEach((k) => {
    if (!prevProps[k]) {
      callback(k, undefined, nextProps[k]);
    }
  });
}

function updateDOM(dom, prevProps, nextProps) {
  compareProps(prevProps, nextProps, (k, prevPropsValue, nextPropsValue) => {
    if (prevPropsValue === nextPropsValue || k === "children") {
      // pass
    } else if (k[0] === "o" && k[1] === "n") {
      const eventName = k.substring(2).toLowerCase();
      if (prevPropsValue) dom.removeEventListener(eventName, prevPropsValue);
      dom.addEventListener(eventName, nextPropsValue);
    } else if (k === "style") {
      compareProps(
        prevPropsValue,
        nextPropsValue,
        (styleKay, prevStyle, nextStyle) => {
          if (prevStyle !== nextStyle) {
            dom[k][styleKay] = nextStyle || "";
          }
        }
      );
    } else if (k in dom) {
      dom[k] = nextPropsValue;
    } else if (prevProps === null || nextProps === false) {
      dom.removeAttribute(k);
    } else {
      dom.setAttribute(k, nextPropsValue);
    }
  });
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

  updateDOM(node, {}, fiber.props);

  return node;
}
let nextUnitOfWork = null;
let workInProcessRoot = null;
let currentRoot = null;
let deletions = null;

function reconcileChildren(fiber, elements) {
  let oldFiber = fiber.alternate && fiber.alternate.child;

  let prevSibling = null;
  let index = 0;
  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;
    const sameType = oldFiber && element && oldFiber.type === element.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: fiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      };
    }

    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: fiber,
        alternate: null,
        effectTag: "PLACEMENT",
      };
    }

    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      fiber.child = newFiber;
    } else {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}

function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }

  const { children } = fiber.props;
  reconcileChildren(fiber, children);

  if (fiber.child) return fiber.child;

  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}

function commitWork(fiber) {
  if (!fiber) return;

  const { dom } = fiber.parent;

  if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
    dom.appendChild(fiber.dom);
  } else if (fiber.effectTag === "DELETION") {
    dom.removeChild(fiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
    updateDOM(fiber.dom, fiber.alternate.props, fiber.props);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitRoot() {
  deletions.forEach(commitWork);
  commitWork(workInProcessRoot.child);
  currentRoot = workInProcessRoot;
  workInProcessRoot = null;

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
    alternate: currentRoot,
  };
  deletions = [];
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

下一步需要支持函数组件

# 第五版

首先我们将

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

改为

```javascript
const App = () => {
  return (
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
};
```

然后代码就报错了。 所以需要一个措施来识别是一个函数还是一个 jsx 对象

```javascript
/** @jsx toyReact.createElement */
const App = () => (
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

toyReact.render(<App />, container);

// 其中 <App /> 会被转化为 toyReact.createElement(App, null)

// 这里第一个参数接收到的就是 App 这个函数， 所以 type 也就是 function 本身
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

```javascript
function performUnitOfWork(fiber) {
  // 这里判断一下 type 是不是一个函数
  const isFunctionComponent = typeof fiber.type === "function";
  // 如果是函数单独处理
  if (isFunctionComponent) {
    updateFunctionCOmponent(fiber);
  } else {
    // 如果不是函数单独处理
    updateHostComponent(fiber);
  }

  if (fiber.child) return fiber.child;

  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}
```

```javascript
// 如果是函数那么吧函数的返回值拿出来当作children
function updateFunctionCOmponent(fiber) {
  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}

// 原始类型和以前处理办法一样
function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }
  reconcileChildren(fiber, fiber.props.children);
}
```

```javascript
function commitDeletion(fiber, domParent) {
  // 判断当当前有没有dom节点
  if (fiber.dom) {
    domParent.removeChild(fiber.dom);
  } else {
    // 如果没有那么递归看看子节点是否存在dom

    commitDeletion(fiber.child, domParent);
  }
}

function commitWork(fiber) {
  if (!fiber) return;

  // 获取父节点
  let domParentFiber = fiber.parent;
  // 如果父节点没有 dom节点那么继续获取父节点
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }

  if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
    domParentFiber.dom.appendChild(fiber.dom);
  } else if (fiber.effectTag === "DELETION") {
    // 同样删除需要额外操作
    commitDeletion(fiber, domParentFiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
    updateDOM(fiber.dom, fiber.alternate.props, fiber.props);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

最后是全体代码

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

function compareProps(prevProps, nextProps, callback) {
  prevProps = prevProps || {};
  nextProps = nextProps || {};
  Object.keys(prevProps).forEach((k) =>
    callback(k, prevProps[k], nextProps[k])
  );

  Object.keys(nextProps).forEach((k) => {
    if (!prevProps[k]) {
      callback(k, undefined, nextProps[k]);
    }
  });
}

function updateDOM(dom, prevProps, nextProps) {
  compareProps(prevProps, nextProps, (k, prevPropsValue, nextPropsValue) => {
    if (prevPropsValue === nextPropsValue || k === "children") {
      // pass
    } else if (k[0] === "o" && k[1] === "n") {
      const eventName = k.substring(2).toLowerCase();
      if (prevPropsValue) dom.removeEventListener(eventName, prevPropsValue);
      dom.addEventListener(eventName, nextPropsValue);
    } else if (k === "style") {
      compareProps(
        prevPropsValue,
        nextPropsValue,
        (styleKay, prevStyle, nextStyle) => {
          if (prevStyle !== nextStyle) {
            dom[k][styleKay] = nextStyle || "";
          }
        }
      );
    } else if (k in dom) {
      dom[k] = nextPropsValue;
    } else if (prevProps === null || nextProps === false) {
      dom.removeAttribute(k);
    } else {
      dom.setAttribute(k, nextPropsValue);
    }
  });
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

  updateDOM(node, {}, fiber.props);

  return node;
}
let nextUnitOfWork = null;
let workInProcessRoot = null;
let currentRoot = null;
let deletions = null;

function reconcileChildren(fiber, elements) {
  let oldFiber = fiber.alternate && fiber.alternate.child;

  let prevSibling = null;
  let index = 0;
  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;
    const sameType = oldFiber && element && oldFiber.type === element.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: fiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      };
    }

    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: fiber,
        alternate: null,
        effectTag: "PLACEMENT",
      };
    }

    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      fiber.child = newFiber;
    } else {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}

function updateFunctionCOmponent(fiber) {
  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}

function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }
  reconcileChildren(fiber, fiber.props.children);
}

function performUnitOfWork(fiber) {
  const isFunctionComponent = typeof fiber.type === "function";

  if (isFunctionComponent) {
    updateFunctionCOmponent(fiber);
  } else {
    updateHostComponent(fiber);
  }

  if (fiber.child) return fiber.child;

  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}

function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom);
  } else {
    commitDeletion(fiber.child, domParent);
  }
}

function commitWork(fiber) {
  if (!fiber) return;

  let domParentFiber = fiber.parent;

  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }

  if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
    domParentFiber.dom.appendChild(fiber.dom);
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParentFiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
    updateDOM(fiber.dom, fiber.alternate.props, fiber.props);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitRoot() {
  deletions.forEach(commitWork);
  commitWork(workInProcessRoot.child);
  currentRoot = workInProcessRoot;
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
    alternate: currentRoot,
  };
  deletions = [];
  nextUnitOfWork = workInProcessRoot;
  requestIdleCallback(workLoop);
}

const toyReact = {
  createElement,
  render,
};

/** @jsx toyReact.createElement */
const App = () => (
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

toyReact.render(<App />, container);
```

# 第六版

目前已经支持函数组建了，那么最后一步就是支持 hooks

```javascript
/** @jsx toyReact.createElement */
const App = () => {
  const [count, setCount] = toyReact.useState(0);

  return <div onClick={() => setCount(count + 1)}>count : {count}</div>;
};
const container = document.getElementById("root");

toyReact.render(<App />, container);
```

这里需要实现一个 useState 函数

```javascript
let hookIndex = null;
let workInProcessesFiber = null;

function updateFunctionComponent(fiber) {
  workInProcessesFiber = fiber;
  hookIndex = 0;
  workInProcessesFiber.hooks = [];

  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}

function useState(init) {
  // 读取旧的 hook 数组
  const oldHook =
    workInProcessesFiber.alternate &&
    workInProcessesFiber.alternate.hooks &&
    workInProcessesFiber.alternate.hooks[hookIndex];

  // 新建一个hook对象
  const hook = {
    // 旧的数据或者初始化的数据
    state: oldHook ? oldHook.state : init,
    queue: [],
  };

  const actions = oldHook ? oldHook.queue : [];
  // 读取当前hook并执行回调函数
  actions.forEach((action) => {
    hook.state = action(hook.state);
  });

  // setState 方法
  const setState = (action) => {
    // 添加到队列里面
    hook.queue.push(action);
    // 这里将 fiberRoot 重新赋值
    workInProcessRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    };
    // 并且复制 nextUnitOfWork 下一个浏览器周期就会以此开始进行更i性能
    nextUnitOfWork = workInProcessRoot;
    deletions = [];
  };

  workInProcessesFiber.hooks.push(hook);
  hookIndex++;
  return [hook.state, setState];
}
```

可以看出， hooks 也是存在 fiber 对象内，因为使用了 index 来确定下标所以 hooks 不允许在 if 内部生命， 因为 hooks 依赖于其出现的顺序

最后是所有代码

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
        typeof child === "string" || typeof child === "number"
          ? createTextElement(child)
          : child
      ),
    },
  };
}

function compareProps(prevProps, nextProps, callback) {
  prevProps = prevProps || {};
  nextProps = nextProps || {};
  Object.keys(prevProps).forEach((k) =>
    callback(k, prevProps[k], nextProps[k])
  );

  Object.keys(nextProps).forEach((k) => {
    if (!prevProps[k]) {
      callback(k, undefined, nextProps[k]);
    }
  });
}

function updateDOM(dom, prevProps, nextProps) {
  compareProps(prevProps, nextProps, (k, prevPropsValue, nextPropsValue) => {
    if (
      prevPropsValue === nextPropsValue ||
      k === "children" ||
      k === "__self" ||
      k === "__source"
    ) {
      // pass
    } else if (k[0] === "o" && k[1] === "n") {
      const eventName = k.substring(2).toLowerCase();
      if (prevPropsValue) dom.removeEventListener(eventName, prevPropsValue);
      dom.addEventListener(eventName, nextPropsValue);
    } else if (k === "style") {
      compareProps(
        prevPropsValue,
        nextPropsValue,
        (styleKay, prevStyle, nextStyle) => {
          if (prevStyle !== nextStyle) {
            dom[k][styleKay] = nextStyle || "";
          }
        }
      );
    } else if (k in dom) {
      dom[k] = nextPropsValue;
    } else if (prevProps === null || nextProps === false) {
      dom.removeAttribute(k);
    } else {
      dom.setAttribute(k, nextPropsValue);
    }
  });
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

  updateDOM(node, {}, fiber.props);

  return node;
}
let nextUnitOfWork = null;
let workInProcessRoot = null;
let currentRoot = null;
let deletions = null;

function reconcileChildren(fiber, elements) {
  let oldFiber = fiber.alternate && fiber.alternate.child;

  let prevSibling = null;
  let index = 0;
  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;
    const sameType = oldFiber && element && oldFiber.type === element.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: fiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      };
    }

    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: fiber,
        alternate: null,
        effectTag: "PLACEMENT",
      };
    }

    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      fiber.child = newFiber;
    } else {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}

let hookIndex = null;
let workInProcessesFiber = null;

function updateFunctionComponent(fiber) {
  workInProcessesFiber = fiber;
  hookIndex = 0;
  workInProcessesFiber.hooks = [];

  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}

function useState(init) {
  const oldHook =
    workInProcessesFiber.alternate &&
    workInProcessesFiber.alternate.hooks &&
    workInProcessesFiber.alternate.hooks[hookIndex];

  const hook = {
    state: oldHook ? oldHook.state : init,
    queue: [],
  };

  const actions = oldHook ? oldHook.queue : [];

  actions.forEach((action) => {
    hook.state = action(hook.state);
  });

  const setState = (action) => {
    hook.queue.push(action);
    workInProcessRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    };
    nextUnitOfWork = workInProcessRoot;
    deletions = [];
  };

  workInProcessesFiber.hooks.push(hook);
  hookIndex++;
  return [hook.state, setState];
}

function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDOM(fiber);
  }
  reconcileChildren(fiber, fiber.props.children);
}

function performUnitOfWork(fiber) {
  const isFunctionComponent = typeof fiber.type === "function";

  if (isFunctionComponent) {
    updateFunctionComponent(fiber);
  } else {
    updateHostComponent(fiber);
  }

  if (fiber.child) return fiber.child;

  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.parent;
  }
}

function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom);
  } else {
    commitDeletion(fiber.child, domParent);
  }
}

function commitWork(fiber) {
  if (!fiber) return;

  let domParentFiber = fiber.parent;

  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }

  if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
    domParentFiber.dom.appendChild(fiber.dom);
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParentFiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
    updateDOM(fiber.dom, fiber.alternate.props, fiber.props);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitRoot() {
  deletions.forEach(commitWork);
  commitWork(workInProcessRoot.child);
  currentRoot = workInProcessRoot;
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
    alternate: currentRoot,
  };
  deletions = [];
  nextUnitOfWork = workInProcessRoot;
  requestIdleCallback(workLoop);
}

const toyReact = {
  createElement,
  render,
  useState,
};

/** @jsx toyReact.createElement */
const App = () => {
  const [count, setCount] = toyReact.useState(0);
  return (
    <div onClick={() => setCount((count) => count + 1)}>count : {count}</div>
  );
};

const container = document.getElementById("root");

toyReact.render(<App />, container);
```

那么看下来和真正的 react 区别在哪里， 首先 react 会通过 key 之类的来跳过未发生变化的节点，而 toyReact 会遍历整棵树， react 使用了事件代理和事件合成， 而 toyreact 使用的是浏览器的原生事件， react 使用了自己写的调度器等


参考资料：https://pomb.us/build-your-own-react/
