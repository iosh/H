---
title: Vue CLI使用scss
date: 2017-11-19 15:56:29
tags: scss
categories: 
	- 教程
---
##  在Vue CLI脚手架中使用scss
今天学习vue当中遇到了如何配置scss的问题经过查证，写出记录。

`第一步`
		npm install sass-loader node-sass
	或者
		yarn add sass-loader node-sass
>  如果node-sass在Windows中安装失败(需要Python和C++构建工具），大家可以使用cnpm单独安装node-sass

`第二步`
在vue文件中
```css
<style  lang='scss'>

#app {

@import  url('./assets/aaa.scss'); // 引入单独的scss文件

font-family: 'Avenir', Helvetica, Arial, sans-serif;

-webkit-font-smoothing: antialiased;

-moz-osx-font-smoothing: grayscale;

text-align: center;

color: #2c3e50;

margin-top: 60px;

}
```
使用sass同理

