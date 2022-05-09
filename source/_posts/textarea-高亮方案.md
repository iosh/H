---
title: textarea 高亮方案
date: 2022-05-09 18:12:40
tags:
---

textarea 标签显示高亮

<!-- more -->

一般用 textarea 来输入大段文字之类的, 需求是用户输入之后, 高亮其中的一些, 一般的需求都是给定文字然后渲染出高亮.而这次的需求是在 textarea 上直接显示. 通过转换思路来完成这个事情.

[太长不看系列,点击这里可以直接看代码](https://codesandbox.io/s/silly-river-m3vjfz?file=/src/App.js)

## 布局

首先准备 dom 布局

```JavaScript
App() {
  return (
    <div className="container">
      <div className="backdrop">
        <div className="highlights"></div>
      </div>
      <textarea></textarea>
    </div>
  );
}
```

container 是容器, 包裹子元素, 并提供定位锚点

backdrop 和 highlights 是显示高亮区域

textarea 就是用户输入区域了

## 样式

```css
.container {
  width: 400px;
  height: 400px;
  border: 1px solid;
  position: relative;
}

.backdrop,
.container textarea {
  width: 400px;
  height: 400px;
  padding: 10px;
  box-sizing: border-box;
  position: absolute;
}

.highlights {
  /*正确处理换行 设置文本颜色为透明*/
  white-space: pre-wrap;
  word-wrap: break-word;
  color: transparent;
}

.backdrop {
  /*高亮要比 textarea 低一层*/
  z-index: 1;
  background-color: #fff;
  overflow: auto;
}

.container textarea {
  /* 设置字体属性, 设置背景色透明*/
  font-family: inherit;
  font-size: 100%;
  line-height: inherit;
  color: inherit;
  background-color: transparent;
  z-index: 2;
}

.backdrop,
.container textarea,
mark {
  font-size: 14px;
}

mark {
  color: transparent;
  /*给一个底色, 这个颜色可以通过附加不同的样式覆盖*/
  background-color: yellow;
}
```

## 添加事件监听

```JavaScript

export default function App() {
  const [value, setValue] = React.useState("");

// 获取元素 dom
  const backdropRef = React.useRef(null);
  const textareaRef = React.useRef(null);
  const highlightRef = React.useRef(null);

  const handleChange = (e) => {
    setValue(e.target.value);
  };

    // 处理滚动事件, 这样滚动是同步的
  const handleScroll = () => {
    if (textareaRef.current && backdropRef.current) {
      const { scrollTop } = textareaRef.current;
      backdropRef.current.scrollTop = scrollTop;
    }
  };

// 这个函数就是用来添加html 标签, 通过正则匹配所有大写字母, 将大写字母转化为 mark 标签
  const highlightsText = (s) => {
    return s.replace(/\n$/g, "\n\n").replace(/[A-Z]/g, "<mark>$&</mark>");
  };

  React.useEffect(() => {
    if (highlightRef.current) {
      highlightRef.current.innerHTML = highlightsText(value);
    }
  });

  return (
    <div className="container">
      <div ref={backdropRef} className="backdrop">
        <div ref={highlightRef} className="highlights"></div>
      </div>
      <textarea
        onScroll={handleScroll}
        ref={textareaRef}
        value={value}
        onChange={handleChange}
      ></textarea>
    </div>
  );
}
```


## 性能优化

这里每次用户输入, 都会触发全量的替换和更新, 可以进行优化. 不过一般而言 textarea 数据量不会很大.

1. 使用非受控组件, 使用非受控组件直接操作dom来更新读取 textarea
2. 优化高亮算法, 可以通过缓存, 比对, 等方法, 也可以使用第三方库, 例如 https://github.com/julmot/mark.js
3. 终极解决办法, 虚拟渲染, 类似于无限滚动一样, 只渲染可视区域内的文字