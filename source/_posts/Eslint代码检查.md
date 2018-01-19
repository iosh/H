---
title: Eslint代码检查
date: 2017-11-28 18:41:12
tags: Eslint
categories: 
	- 教程
---
### 前言
使用Eslint检查代码规范是一个很好的习惯，基本上只要符合Eslint代码检查，那么编译过程很少会看到报错(除非逻辑错误)

> 在此纠正一个观点，单引号双引号，结尾加不加分号，都是可以的，根据团队或者个人喜好即可。

#### 第一步
    npm install -g eslint
是否全局安装看个人喜好吧

为了避免繁琐的配置过程，我们使用社区比较流行的

    npm install -g install-peerdeps
    install-peerdeps --dev eslint-config-airbnb


上面安装三个依赖install-peerdeps、install-peerdeps、eslint-config-airbnb
#### 第二步
然后在项目根目录下建立一个`.eslintrc`文件，windows下无新建.开头的文件夹，请使用vs code建立。

然后在里面写上

```json
{
  "extends": "airbnb",
  "env": {
    "browser": true, // 代码运行环境，browser是浏览器，这样使用document之类的就不会报未定义了
    "node": true,
    "jest": true
  },
  "rules": {
    "react/jsx-filename-extension": [
      1,
      { // 允许在js文件里面写jsx
        "extensions": [
          ".js",
          ".jsx"
        ]
      }
    ],
    "linebreak-style": "off", // 关闭windows和linux系统下换行符不一致
    "react/prop-types": 0, // 关闭强制type-props验证
    "no-underscore-dangle": 0,// 关闭前下划线检测
    "react/prefer-stateless-function": 0, // 关闭class-function组件检测
    "jsx-a11y/anchor-is-valid": [ "error", {
      "components": [ "Link" ],
      "specialLink": [ "to" ]
    }], // 修改Link组件警告
    "react/no-array-index-key": 0 // 关闭不允许使用index作为key
  },
  "globals": { // 全局变量使用这里规定的全局变量不会报错
    "var1": true, // 允许重写
    "var2": false // 不允许重写
  }
}
```

然后就可以畅快的写代码了，上面规则因人而异，不习惯了可以改。

