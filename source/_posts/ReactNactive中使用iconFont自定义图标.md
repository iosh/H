---
title: ReactNactive中使用iconFont自定义图标
date: 2018-04-11 19:59:14
tags: ReactNative
categories: 
	- 教程
---

在 reactNative 使用自定义 iconFont 自定义图标

<!-- more -->

因为在 ReactNative 中无法使用 svg 格式文件（其实可以，不过很麻烦），使用 png 图片又没办法简单的切换图标颜色。

所以自定义图标还是很有用的，下面介绍一种比较简单的方法

需要用到的东西：

1.[iconfont](http://iconfont.cn/) 阿里巴巴矢量图平台

在上面的阿里巴巴矢量图平台选择好图标加入购物车，然后点击购物车点击下载代码。

之后需要登录，登录之后就可以下载了

2.[react-native-vector-icons](https://github.com/oblador/react-native-vector-icons) 用来显示自定义图标，本身自带了很多图标，基本也够用。

    npm i react-native-vector-icons
    react-native link



那么接下来需要需要两步：

第一步：[https://github.com/bob-chen/react-native-iconfont-mapper](https://github.com/bob-chen/react-native-iconfont-mapper)将这个项目克隆到本地，之后需要安装 Python 环境，环境安装完毕之后需要安装依赖

    pip install fonttools

然后将在阿里下载的 iconfont 压缩包里面的 iconfont.ttf 文件放到 react-native-iconfont-mapper 根目录下，之后输入

    python iconfont-mapper.py iconfont.ttf font.js

他会生成一个 font.js 的 js 文件里面是这个样子

```JavaScript
var map = {"arrow":"62976","checked":"62977","checked-s":"62978","tag-svip":"62995"};

module.exports = (name)=>String.fromCharCode(map[name]);

module.exports.map = map;
```

将里面的 map 对象复制一下

之后在项目里面新建一个 js 文件比如 Icon.js

```JavaScript
import { createIconSet } from 'react-native-vector-icons';

var map = {
    "arrow":62976,
    "checked":62977,
    "checked-s":62978,
    "tag-svip":62995
    };

// 上面的map是脚本帮我们提取出来的记得将数字的引号去掉
// 然后createIconSet的第二个参数是字体名称，我被这个坑了好久，首先说一下，阿里巴巴下载来的字体名称叫做'iconfont'
// 确认字体叫什么名字很简单，ios直接查看详细信息，windows也差不多
// createIconSet第三个参数是字体文件名，别写错了
const iconSet = createIconSet(map, 'iconfont', 'aliiconfont.ttf');

export default iconSet;

// 使用就很简单了
import React from 'react'
import Icon from './Icon'

function Abcd () {
    return <Icon name="arrow">
}


```

接下来还是这个 ttf 文件：

ios 使用 xcode 打开 ReactNative 项目将 iconfont.ttf 改个名比如改成 aliiconfont.ttf 然后将文件放入 文件夹内，因为上面已经 react-native link 过了，所以文件夹内有好几个 ttf 文件都是 react-native-vector-icons 包自带的,
之后在`info.plst`下 Fonts provided by application 属性，照着其他的把填上去 aliiconfont.ttf
