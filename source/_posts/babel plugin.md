---
title: babel plugin
date: 2020-05-23 19:56:17
tags: babel
---

learning how to create babel plugin

<!-- more -->

babel 主要处理流程分为三步:

`解析 parse` -> `转换 transform` -> `生成 generate`

目前 babel 的三个步骤工作分别由:

`@babel/parser` -> `@babel-plugin` -> `@babel/generator`

# 目标

编写一个将 const 和 let 转化为 var 的 babel-plugin

# 准备工作

安装 `@babel/parser`

```bash
yarn add @babel/parser @babel/traverse @babel/types
```

# 施工 🚧

## 解析 JavaScript 字符

首先需要将 JavaScript 字符串转化为 AST(`Abstract Syntax Tree`)

```js
const parse = require("@babel/parser").parse;
const tree = parse("const a = () => { return b}");
```

这样就可以得到一颗 `Abstract Syntax Tree`, parse 有很多 option , 有详细的文档供介绍.

得到的结果可以在本地打印,或者在 [astexplorer.net](https://astexplorer.net/)进行查看.

通过观察树,可以发现 `const a` 这部分被定义为一个 `variableDeclaration` Node.

而 `const` `let` `var` 之间的差距,就仅仅是 Node 的 `kind` 值的不同

## 修改

那么就是使用 `@babel/traverse` 插件匹配 `variableDeclaration`, 然后修改 `kind` 的值就可以了.

```js
const ast = parse("const a = () => { return b}");
traverse(ast, {
  enter(path) {
    if (t.isVariableDeclaration(path.node)) {
      if (path.node.kind !== "var") {
        path.node.kind = "var";
      }
    }
  },
});

// 生成 JavaScript 代码字符串

const code = generate(ast);
```

# 插件制作

新建一个文件 `transform-to-var.js`

```ts
module.exports = function ({ types: t }) {
  return {
    visitor: {
      VariableDeclaration(path) {
        if (path.node.kind !== "var") {
          path.node.kind = "var";
        }
      },
    },
  };
};
```

安装 babel plugin 测试程序和测试框架

```bash
yarn add babel-plugin-tester jest
```

新建一个 `transform-to-var.test.js`

```js
const pluginTester = require("babel-plugin-tester").default;
const myPlugin = require("./transform-to-var");
console.log(myPlugin);
pluginTester({
  plugin: myPlugin,
  pluginName: "transform to var",
  title: "trabsforn const let to var",
  tests: {
    "const a = 10: var a = 10": {
      code: "const a = 10;",
      output: "var a = 10;",
    },
    "let a = 10: var a = 10": {
      code: "let a = 10;",
      output: "var a = 10;",
    },
    "not change code": "var a = 10;",
  },
});
```

运行测试

```bash
yarn jest
```
