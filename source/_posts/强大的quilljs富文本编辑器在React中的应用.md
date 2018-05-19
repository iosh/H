---
title: 强大的quilljs富文本编辑器在React中的应用
date: 2018-5-19 12:42:09
tags: React
---

在 React 中富文本编辑器框架有 Facebook 开源的著名 [draft-js](https://github.com/facebook/draft-js) 很厉害，但是他只是一个基础框架，并不可以直接拿来使用，像知乎的编辑器就是基于他实现的，我这种弱鸡不会，只能找一个简单的富文本编辑器来使用，这里我看到了Quill
<!-- more -->

Quill 在 GitHub 上有1.8w个赞，功能丰富，React 也有一个移植版叫做 React-Quill 但是我在他提供的例子上发现一个严重的 bug 就是在每一行的第一个字无法使用中文，比如另起一行输入

	你好

实际上他会显示

	n你好

会多出来一个字母，而且 React-Quill 也只有不到2k的赞，所以只能把目光转向 Quilljs 

他的用法很简单

```jsx
import React from "react";
import Quill from "quill";
import { render } from "react-dom";
import 'quill/dist/quill.snow.css'

class App extends React.Component {
  state = {
    value: ""
  };
  componentDidMount() {
    // 配置项，在Quill 官网上有详细说明
    const options = {
      debug: "warn",
      theme: "snow"
    };
    // 实例化 Quill 并且储存在实例原型上
    this.editor = new Quill("#editor", options);
    // 实现输入受控，从state中读取html字符串写入编辑器中
    const { value } = this.state;
    // 判断value中是否有值，如果有那么就写入编辑器中
    if (value) this.editor.clipboard.dangerouslyPasteHTML(value);
    // 设置事件，change事件，
    this.editor.on("text-change", this.handleChange);
  }
  handleChange = () => {
    // change 事件将HTML字符串更新到state里面，
    this.setState({
      value: this.editor.root.innerHTML,
      mediaVisbile: false
    });
  };
  render() {
    return <div id="editor" />;
  }
}

render(<App />, document.getElementById("root"));
```

这样就实现了一个简单的富文本编辑器，接下来可以定制 toolbar 工具栏实现一些自己想要的功能
Quill 通过 id 选择器来实现 HTML 匹配，这是最简单的方法了

所以定制工具栏方法十分简单

```jsx
import React from "react";
import Quill from "quill";
import { render } from "react-dom";
import "quill/dist/quill.snow.css";

class App extends React.Component {
  state = {
    value: ""
  };
  componentDidMount() {
    // 配置项，在Quill 官网上有详细说明
    const options = {
      debug: "warn",
      theme: "snow",
      modules: { // 自定义 toolbar 填写这个属性， 值写上 div 的 id
        toolbar: "#toolbar"
      }
    };
    // 实例化 Quill 并且储存在实例原型上
    this.editor = new Quill("#editor", options);
    // 实现输入受控，从state中读取html字符串写入编辑器中
    const { value } = this.state;
    // 判断value中是否有值，如果有那么就写入编辑器中
    if (value) this.editor.clipboard.dangerouslyPasteHTML(value);
    // 设置事件，change事件，
    this.editor.on("text-change", this.handleChange);
  }
  handleChange = () => {
    // change 事件将HTML字符串更新到state里面，
    this.setState({
      value: this.editor.root.innerHTML,
      mediaVisbile: false
    });
  };
  render() {
    return (
      <div style={{ backgroundColor: "#fff" }}>
        <div id="toolbar" style={{ display: "flex", alignItems: "center" }}>
          <select className="ql-size" defaultValue="small">
            <option value="small">小</option>
            <option value="large">中</option>
            <option value="huge">大</option>
          </select>
          {/* <select className="ql-header" defaultValue="false">
            <option value="1">1</option>
            <option value="2">2</option>
            <option value="3">3</option>
            <option value="false">x</option>
          </select> */}
          <button className="ql-bold" />
          <select className="ql-color" defaultValue="">
            <option value="#f5222d" />
            <option value="#fa541c" />
            <option value="#fa8c16" />
            <option value="#faad14" />
          </select>
          <select className="ql-background" defaultValue="">
            <option value="#f5222d" />
            <option value="#fa541c" />
            <option value="#fa8c16" />
            <option value="#faad14" />
          </select>
		  <button
            style={{ width: "80px" }}
            onClick={() =>
              console.log("这里可以通过读取state来拿到所有的html字符串")
            }
          >
            提交
          </button>
        </div>
        <div id="editor" />
      </div>
    );
  }
}

render(<App />, document.getElementById("root"));

```

Quill 的所有配置项 [文档在这里](https://quilljs.com/docs/modules/toolbar/)
简单到只需要看文档上原有的名字叫什么然后新建一个div 给个 id=ql-配置名称即可

Quill 提供了丰富多样的 api 从内容获取，到添加样式之类的应有尽有，可以配合起来可以实现比较多的功能
