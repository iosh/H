---
title: ReactNative遇到的报错
date: 2018-02-14 19:22:19
tags: ReactNative
---

## 最近在学习ReactNative开发,上来就遇到报错,所以写个博客吧自己遇到的报错都总结一下
<!-- more -->
### 报错一error in opening zip file
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

打开路径*C:\Users\此处是你电脑的用户名\.gradle\wrapper\dists
删除dists目录下的所有文件(其实就是gradle-2.14.1-all)多数是因为网络原因gradle的压缩包没下载完全
导致解压失败,删掉之后他会重新下载解压.

### 报错二Your project path contains non-ASCII characters.
具体报错内容忘了复制,报错内容机翻都能看懂.
就是路径中不能有中文.换个目录就好了


### 报错三Could not find tools.jar

```
* What went wrong:
Execution failed for task ':app:compileDebugJavaWithJavac'.
> Could not find tools.jar. Please check that E:\java contains a valid JDK installatio


```
解决办法:
是java jdk安装问题或者环境变量问题,如果安装了就控制面板卸载java系列,重新安装一路下一步不要更改安装路径.重新安装一下就好了

### 报错四No connected devices!

```

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: No connected devices!

```
解决办法:
我是使用真机来学习的,但是插上电脑之后我手机一直没被电脑识别(此处需要豌豆荚等xxx手机助手,如果这些软件能发现手机说明你真机连接无误).我手机连豌豆荚都发现不了他.可能是手机接口或者数据线问题吧.
那就使用Android Studio自带的模拟器试试.
https://www.genymotion.com/fun-zone/

这里可以下载个人版本genymotion比as自带的启动快好用
