---
title: ReactNative遇到的报错
date: 2018-02-14 19:22:19
tags: ReactNative
---

## 最近在学习 ReactNative 开发,上来就遇到报错,所以写个博客吧自己遇到的报错都总结一下

<!-- more -->

### 报错一 error in opening zip file

```
Scanning folders for symlinks in E:\练习项目\HAPP\node_modules (22ms)
JS server already running.
Building and installing the app on the device (cd android && gradlew.bat installDebug)...
Unzipping C:\Users\H\.gradle\wrapper\dists\gradle-2.14.1-all\8bnwg5hd3w55iofp58khbp6yv\gradle-2.14.1-all.zip to C:\Users\H\.gradle\wrapper\dists\gradle-2.14.1-all\8bnwg5hd3w55iofp58khbp6yv
Exception in thread "main" java.util.zip.ZipException: error in opening zip file
        at java.util.zip.ZipFile.open(Native Method)
        at java.util.zip.ZipFile.<init>(Unknown Source)
        at java.util.zip.ZipFile.<init>(Unknown Source)
        at java.util.zip.ZipFile.<init>(Unknown Source)
        at org.gradle.wrapper.Install.unzip(Install.java:159)
        at org.gradle.wrapper.Install.access$500(Install.java:26)
        at org.gradle.wrapper.Install$1.call(Install.java:69)
        at org.gradle.wrapper.Install$1.call(Install.java:46)
        at org.gradle.wrapper.ExclusiveFileAccessManager.access(ExclusiveFileAccessManager.java:65)
        at org.gradle.wrapper.Install.createDist(Install.java:46)
        at org.gradle.wrapper.WrapperExecutor.execute(WrapperExecutor.java:126)
        at org.gradle.wrapper.GradleWrapperMain.main(GradleWrapperMain.java:61)
```

这个问题的解决办法经:

打开路径\*C:\Users\此处是你电脑的用户名\.gradle\wrapper\dists
删除 dists 目录下的所有文件(其实就是 gradle-2.14.1-all)多数是因为网络原因 gradle 的压缩包没下载完全
导致解压失败,删掉之后他会重新下载解压.

### 报错二 Your project path contains non-ASCII characters.

具体报错内容忘了复制,报错内容机翻都能看懂.
就是路径中不能有中文.换个目录就好了

### 报错三 Could not find tools.jar

```
* What went wrong:
Execution failed for task ':app:compileDebugJavaWithJavac'.
> Could not find tools.jar. Please check that E:\java contains a valid JDK installatio


```

解决办法:
是 java jdk 安装问题或者环境变量问题,如果安装了就控制面板卸载 java 系列,重新安装一路下一步不要更改安装路径.重新安装一下就好了

### 报错四 No connected devices!

```

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: No connected devices!

```

解决办法:
我是使用真机来学习的,但是插上电脑之后我手机一直没被电脑识别(此处需要豌豆荚等 xxx 手机助手,如果这些软件能发现手机说明你真机连接无误).我手机连豌豆荚都发现不了他.可能是手机接口或者数据线问题吧.
那就使用 Android Studio 自带的模拟器试试.
https://www.genymotion.com/fun-zone/

这里可以下载个人版本 genymotion 比 as 自带的启动快好用

### 报错五 Unsupported top level event type "topTouchStart" dispatched

```
Unsupported top level event type "topTouchStart" dispatched

```

出现这个报错的时候,我是从 react-navigation 网站上复制了一段代码,然后保存然后报错了,
我谷歌搜索发现很多人遇到了这个问题,说 react-native 一直有这个问题,只能降级才能解决,后来经我测试重新 react-native run-android 一下就可以了,报错一次重新编译一下.这个错就会被解决

### 报错六 While resolving module `react-native-vector-icons

今天在使用 react-native-vector-icons 这个库的时候,运行程序之后报错了

```
While resolving module `react-native-vector-icons/这里是你具体引入的名称`, the Haste package `react-native-vector-icons` was found.
```

react 版本是 16.2
react-native 版本是 0.52

最后经多方查询找到解决方案
linux/macos 系统在终端至工程目录输入一下命令

```
rm ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json

```

windows 系统只能手动打开上面那个路径然后删掉最后面的那个 package.json

然后重新编译并运行即可
