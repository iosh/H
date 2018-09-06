title: 常见POST提交数据方式
date: 2018-09-06 18:36:03
tags: HTTP
---

学习一点新知识

<!-- more -->
# POST提价数据方式

HTTP 请求定义了多种请求方式，其中 POST 多用于更新和发送数据，是极为常用的一种请求方式，HTTP协议是建立在 TCP/IP 协议之上的应用层规范，规范将 HTTP 请求分为三个从步骤：


```
// 来自MDN文档

<method><request-URL><version>
<headers>
<entity-body>

<请求方法><请求路径><协议版本>
<请求头>
<请求体>

例如：

POST /user/login HTTP/1.1
Host: example.org
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

say=Hi&to=Mom

```


其中 Content-Type: application/x-www-form-urlencoded

定义了请求体中的数据格式，用于告知服务端该次请求请求体中数据所使用的格式

常见格式有四种:

## application/x-www-form-urlencoded

`application/x-www-form-urlencoded` 数据被编码为以`&`分隔的键-值对，同时以`=`分隔键值对，非字母或数字会被转换，这种数据类型是不支持二进制数据传输的
例如：

```
// 来自MDN文档
POST / HTTP/1.1
Host: foo.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

say=Hi&to=Mom

上面的数据转换成对象就是
{
  say: 'Hi',
  to:'Mom
}
```
这种书传输是非常常见的格式，例如 JQuery 中的 Ajax 默认使用的就是这种格式，服务端处理起来也比较简单

## multipart/form-data

这种数据格式`multipart/form-data`和上面的`application/x-www-form-urlencoded`，都是浏览器原生支持的，例如使用原生的 `<from>` 表单提交数据（通过指定enctype 属性类型）例如：
```
// 来自MDN
POST /test.html HTTP/1.1 
Host: example.org 
Content-Type: multipart/form-data;boundary="boundary" 

--boundary 
Content-Disposition: form-data; name="field1" 

value1 
--boundary 
Content-Disposition: form-data; name="field2"; filename="example.txt" 

value2

```

首先生成了一个 `boundary` 用于分隔不同的字段，一般情况下都比较长为了和正文进行分隔
例如

```
--boundary 
Content-Disposition: form-data; name="field1" 

使用 boundary 进行分隔
内容描述信息： Content-Disposition: form-data; 如果是传输文件那么Content-Disposition: form-data;之后还会有文件类型信息例如

Content-Disposition: form-data; name="userAvatar"; filename="cat.png"  
Content-Type: image/png

上面表明是一张图片，图片名称为cat.png，回车之后是表明文件类型


```

一般情况下这种格式可以用来上传图片

## application/json

JSON格式应该都不陌生

    JSON 是一种和语言无关的数据格式，来源于 JavaScript， 许多编程语言都包含了生成和解析 JSON 格式数据的代码 --- wiki

例如在 Axios 请求库中，默认使用的就是 JSON 字符串

```javascript
import axios from 'axios'

axios.post('/', {name: 'bob', age: 30})

```

# CORS 跨域资源共享

    CORS 跨域资源共享机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行，浏览器支持在 API 容器中使用 CORS ，以降低跨域 HTTP 请求带来的风险。 --- MDN


跨域资源共享标准允许在下列场景中使用跨域 HTTP 请求：

- 通过请求发起的跨域 HTTP 请求
- Web字体(css 中通过 `@font-face` 使用跨域字体资源）
- WebGL贴图
- 使用 drawImage 将 images/video 画面绘制到 canvas
- 样式表
- Scripts(未处理异常)


规范要求，对那些可能对服务器数据产生副作用的HTTP方法，浏览器必须使用 `OPTIONS` 方法发起一个预检请求，从而获知服务器是否允许该跨域请求，服务器允许之后，才发起实际的 HTTP 请求，在预检请求返回中，服务器端也可以通知客户端，是否需要携带身份凭证


## 简单请求

某些请求不会触发 CORS 预检请求，这样的请求被称为`简单请求`，若请求满足所有下述条件，则该请求可以被视为`简单请求`.

- 使用下列方法之一
  - GET
  - HEAD
  - POST

- Fetch 规范定义了[对CORS安全的首部字段集合](https://fetch.spec.whatwg.org/#cors-safelisted-request-header),不得人为设置该集合之外的其他首部字段，该集合为
  - ACCEPT（用于告知客户端可以处理的内容类型）
  - Accept-Language（用于告知客户端可以处理的自然语言）
  - Content-Language（用于说明访问者希望采用的语言或语言组合）
  - Content-Type (请求体的类型说明，并且该字段有额外限制）
  - DPR（用于告知客户端当前设备的像素比率）
  - Save-Data
  - Viewport-Width
  - Width

- Content-Type 的值被限定为以下三种之一
  - text/plain（普通文本）
  - multipart/form-data
  - application/x-www-form-urlencoded


满足上述条件的都被视为`简单请求`



## 预检请求

当不满足简单请求的条件时，浏览器就会使用 `OPTIONS` 方法发起一个预检请求到服务器，并且根据服务器响应来确定服务器是否允许CORS跨域

当满足以下述任一条件时，就会首先发送预检请求：

- 当请求方法为以下任一 HTTP 方法
  - PUT
  - DELETE
  - CONNECT
  - OPTIONS
  - TRACE
  - PATCH
- 设置了简单请求上面列出的其他不在范围内的首部字段
- Content-Type 的值不属于下列之一:
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain


## 附带身份凭证的请求

Fetch 与 CORS 的一个特性是，可以基于 HTTP cookies 和 HTTP 认证信息发送身份凭证，一般而言对于跨域请求，浏览器都不会发送身份凭证信息，如果要发送凭证信息，需要进行设置

例如 Axios 需要设置： withCredentials: true

对于附带身份凭证的请求，服务器不得设置 Access-Control-Allow-Origin 的值为“*”

这是因为请求中携带了 cookie 信息，如果设置了 Access-Control-Allow-Origin 的值为“*”，请求会失败，另外响应首部中也懈怠了 Set-Cookie 字段，尝试对 Cookie 进行修改，如果操作失败，将会跑出异常

## HTTP 响应首部字段

- Access-Control-Allow-Origin

响应首部中可以携带一个 Access-Control-Allow-Origin 字段

其值制定了允许访问该资源的外域 URL ，对于不需要携带身份凭证的请求，服务器可以指定该字段的值为通配符，表示允许来自所有域的请求

例如，下面的字段值将允许来自 http://mozilla.com 的请求：

  Access-Control-Allow-Origin: http://mozilla.com


- Access-Control-Expose-Headers

Access-Control-Expose-Headers 响应头让服务器把允许浏览器访问的头放入白名单例如：

  Access-Control-Expose-Headers: POST, GET, PUT, DELETE

- Access-Control-Max-Age

  Access-Control-Max-Age 头制定了 prefight 请求的结果能够被缓存多久

  Access-Control-Max-Age: 600
  单位是秒，表明预检请求可以被缓存 600 秒

- Access-Control-Allow-Credentials

  响应头部表示是否可以将对请求的响应暴露给页面，返回 true 则可以，其他值均不可以
  

- Access-Control-Allow-Methods

  Access-Control-Allow-Methods 首部字段用于预检请求的响应，其指明了实际请求所允许使用的 HTTP 方法


- Access-Control-Allow-Headers

  Access-Control-Allow-Headers 首部字段用于检测预检请求的响应，其指明了实际请求中允许携带的首部字段


  
  








































































