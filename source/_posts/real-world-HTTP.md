---
title: real world HTTP
date: 2022-11-28 21:10:31
tags: http
---

详解 HTTP 协议基础与 Go 语言实现

<!-- more -->


# HTTP/1.0 的语法：4个基本元素

- 方法和路径
- 首部
- 主体
- 状态码


HTTP协议自 1990 年至今已经成为当代互联网基础，并且仍然在不断升级。

HTTP 的默认端口是 80， 端口基本上是用来确定其用途的， Web 或者 FTP 等客户端如果无特别指定那么默认使用80（HTTPS 默认443）， 但是可以通过指定不同的端口来运行，客户端通过地址+指定端口号的方式来访问服务。


##  MIME类型

目前广泛普及的网站大多数以 HTML文件为基础，如今使用 JavaScript 和样式表来提供丰富的用户体验是必不可少的，浏览器管理者个文件类型对应的操作，用来标识文件类型的标识符就是 MIME 类型。

现在的Web服务器在发送HTML时， 会在服务器返回的响应头中设置 MIME 类型

```text
Content-Type: text/html; charset=utf-8
```

## 方法

HTTP/1.0中发送请求时的GET部分就是方法，1992年的版本中提出了许多方法 GET HEAD POST PUT DELETE


## URL 国际化
URL 域名一开始只支持英文，数字以及连字符， 但是2003年确认了用来标识国际化域名的 Punycode 编码规范，之后使用该规范的浏览器就支持国际化域名了。


## GET 请求时的主体

在HTTP的方法中有一些方法是不希望包含主体的，例如使用 GET方法时候通常使用查询字符串， 但是实际上你也可以使用 curl 来为 GET方法添加请求体

```bash
curl -X get --data "hello world" http://localhost:8888
```

但是这并不是RFC推荐的做法，在任何一个HTTP请求消息都可以包含消息主体， 但是服务器可以根据实现拒绝接受。并且 RFC7231 中强调了只有 HTTP/1.1 中添加的 TEACE 方法不可以包含请求体。


# HTTP/1.0的语义：浏览器基本功能的背后

## 使用 x-www-form-urlencoded 发送表单

通过 HTML 的 form 标签可以发送表单。在 curl 中为：

```bash
curl --http1.0 -d title="the art of community" -d author="Jono Bacon" https://localhost:9999
```
这里 curl 命令会设置首部 Content-Type：application/x-www-form-urlencoded 这个时候主体就会变成使用等号拼接的字符串：

```txt
title=the art of community*author=Jono Bacon
```
如果原文中有等号和&符号就需要会进行转义。分别转化为 %3d 和 %26， 空格会转化为 %20 。


## 使用 multipart/form-data 发送文件

在 HTML 表单中可以选择 multipart/form-data这一编码格式， 比较复杂不过可以发送文件。在使用 x-www-form-urlencoded 的情况下，名称与内容一一对应，而在使用multipart/form-data 的情况下，每个项目会额外的添加元信息作为标签，以标识当前数据的格式，方便服务器进行读取。

## 内容协商

由于服务器和客户端是分开开发和维护的， 所以二者的格式和设置并不总是一致的，为了优化通信方法， 服务器和客户端在一个请求中共享彼此的最有设置， 这种结构就是内容协商。
一般情况下会在头部字段中表面可以处理的 MIME 类型，显示语言，字符集，和压缩类型。

