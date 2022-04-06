---
title: koa 源码
date: 2022-04-06 14:20:15
tags:
---

koa 源码学习

<!-- more -->

# hello koa

```JavaScript
const Koa = require('koa');
const app = new Koa();

// response
app.use(ctx => {
  ctx.body = 'Hello Koa';
});

app.listen(3000);
```

简单几行代码就可以运行一个 koa 服务. 使用 async await 的用法是这样

```JavaScript
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
```

首先使用 new 来初始化一个新的 koa 对象

然后使用 `use` 来加载各个中间件, 最后使用 listen 监听端口.

## listen

这里先看简单的 listen , 随后在学习 use.

```JavaScript
 listen (...args) {
    debug('listen')
    const server = http.createServer(this.callback())
    return server.listen(...args)
  }
```

简单的几行代码, 可以看到使用 node http 模块创建一个 http 服务器, 将 Application 类的 callback 作为参数创建了一个监听指定端口的服务器.

这里 [debug](https://www.npmjs.com/package/debug) 是一个包可以在 node 或者浏览器中作为控制台输出库.

[http.createServer](http://nodejs.cn/api/http.html#httpcreateserveroptions-requestlistener) 用来创建一个 http 服务器, 从文档得知函数基础使用如下.

```JavaScript
const http = require('http');

// 创建本地服务器来从其接收数据
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!'
  }));
});

server.listen(8000);

```

那么 koa 在这里传入的 `this.callback` 必然是返回一个处理 `req` `res` 的函数.

### callback

callback 的内容如下.

```JavaScript
  callback () {
    // 组合所有中间件, 这里稍后再看
    const fn = compose(this.middleware)
    // this 是 Application 继承于node event 的一个类, 这里判断有没有监听 error 的事件,如果没有就默认添加一个
    if (!this.listenerCount('error')) this.on('error', this.onerror)

    // 这就是返回一个 createServer 签名的函数
    const handleRequest = (req, res) => {
        // 创建 context
      const ctx = this.createContext(req, res)
      // 最终由此函数调用res.send 来返回内容
      return this.handleRequest(ctx, fn)
    }

    return handleRequest
  }
```

### createContext

```JavaScript
    /**
     *  用来创建一个 context, 本质上就是一个存放当前请求各种信息和一些方法的对象
    */
  createContext (req, res) {
      // 基础的context来自 context.js 最后看
    const context = Object.create(this.context)
    // 同上来自 request.js
    const request = context.request = Object.create(this.request)
    // 同上来自 response.js
    const response = context.response = Object.create(this.response)
    context.app = request.app = response.app = this
    context.req = request.req = response.req = req
    context.res = request.res = response.res = res
    request.ctx = response.ctx = context
    request.response = response
    response.request = request
    context.originalUrl = request.originalUrl = req.url
    context.state = {}
    return context
  }

```

### handleRequest

```JavaScript
  handleRequest (ctx, fnMiddleware) {
    const res = ctx.res
    // 这里默认状态码设置为 404, 2014年经过讨论这里默认值设置为404
    res.statusCode = 404

    // 这是一个默认的 error 处理函数
    const onerror = err => ctx.onerror(err)
    const handleResponse = () => respond(ctx)
    // on-finished 是一个包,在http异常或者结束的时候都会调用回调函数
    onFinished(res, onerror)
    // 这里经过中间件处理ctx, 然后调用 respond 函数返回浏览器结果
    return fnMiddleware(ctx).then(handleResponse).catch(onerror)
  }

```

### respond

```JavaScript
function respond (ctx) {
    // 可以设置 ctx.respond 为 false 来绕过koa的respond函数
  // allow bypassing koa
  if (ctx.respond === false) return

  if (!ctx.writable) return

  const res = ctx.res
  let body = ctx.body
  const code = ctx.status
    // statuses 是一个状态的枚举包 https://www.npmjs.com/package/statuses
  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null
    return res.end()
  }

  // HEAD 请求 https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD
  if (ctx.method === 'HEAD') {
    if (!res.headersSent && !ctx.response.has('Content-Length')) {
      const { length } = ctx.response
      if (Number.isInteger(length)) ctx.length = length
    }
    return res.end()
  }

  // status body
  if (body == null) {
    if (ctx.response._explicitNullBody) {
      ctx.response.remove('Content-Type')
      ctx.response.remove('Transfer-Encoding')
      ctx.length = 0
      return res.end()
    }
    if (ctx.req.httpVersionMajor >= 2) {
      body = String(code)
    } else {
      body = ctx.message || String(code)
    }
    if (!res.headersSent) {
      ctx.type = 'text'
      ctx.length = Buffer.byteLength(body)
    }
    return res.end(body)
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body)
  if (typeof body === 'string') return res.end(body)
  if (body instanceof Stream) return body.pipe(res)

  // body: json
  body = JSON.stringify(body)
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body)
  }
  res.end(body)
}

```

## use

从下面代码可以看到 use 的实现非常简单, 就是判断 fn 是不是函数, 如果是的话放入 middleware 数组内.并且在 `createServer` 的 [callback](#callback) 函数中可以看到使用了一个 `compose` 函数处理了这个 middleware 数组

```JavaScript
  use (fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!')
    debug('use %s', fn._name || fn.name || '-')
    this.middleware.push(fn)
    return this
  }
```

### compose middleware

see: [callback](#callback) 函数

```JavaScript
const fn = compose(this.middleware)
```

see: [handleRequest](#handleRequest) 函数

```JavaScript
// fnMiddleware 就是上面的 fn
fnMiddleware(ctx).then(handleResponse).catch(onerror)
```

这里的 `compose` 是使用的一个 `koa-compose` 的包源代码也很简单

```JavaScript
function compose (middleware) {
    // 判断是不是数组
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
    // 使用for of 可以遍历可迭代对象 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols
    // 判断其中的每一项是否都为函数, 如果不是那么抛出异常
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

上面代码的最终含义就是

假如有一个 middleware 数组值为 [fn1, fn2,fn3,fn4]
那么经过这个 compose 函数得到的最终结果为

```javascript
Promise.resolve(
  fn1(
    context,
    Promise.resolve(
      f2(
        context,
        Promise.resolve(
          f3(context, Promise.resolve(f4(context, Promise.resolve())))
        )
      )
    )
  )
);
```

整个 koajs 的源码都很简单, 本质上就是实现了一个中间件模型,通过组合不同的中间件,来解析和处理返回.

# @koa/cors

```JavaScript
const Koa = require('koa');
const cors = require('@koa/cors');

const app = new Koa();
app.use(cors());
```

上面已经看了这个 use 的原理, cors 源码也很简单, 本质上就是收到 `OPTIONS` 请求之后添加对应的响应头.

```JavaScript

function(options) {
  const defaults = {
    allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH',
    secureContext: false,
  };

  options = {
    ...defaults,
    ...options,
  };

  // 默认值处理
  if (Array.isArray(options.exposeHeaders)) {
    options.exposeHeaders = options.exposeHeaders.join(',');
  }

  if (Array.isArray(options.allowMethods)) {
    options.allowMethods = options.allowMethods.join(',');
  }

  if (Array.isArray(options.allowHeaders)) {
    options.allowHeaders = options.allowHeaders.join(',');
  }

  if (options.maxAge) {
    options.maxAge = String(options.maxAge);
  }

  options.keepHeadersOnError = options.keepHeadersOnError === undefined || !!options.keepHeadersOnError;

  return async function cors(ctx, next) {
    // If the Origin header is not present terminate this set of steps.
    // The request is outside the scope of this specification.
    const requestOrigin = ctx.get('Origin');

    // Always set Vary header
    // https://github.com/rs/cors/issues/10
    ctx.vary('Origin');

    if (!requestOrigin) return await next();

    let origin;
    if (typeof options.origin === 'function') {
      origin = options.origin(ctx);
      if (origin instanceof Promise) origin = await origin;
      if (!origin) return await next();
    } else {
      origin = options.origin || requestOrigin;
    }

    let credentials;
    if (typeof options.credentials === 'function') {
      credentials = options.credentials(ctx);
      if (credentials instanceof Promise) credentials = await credentials;
    } else {
      credentials = !!options.credentials;
    }

    const headersSet = {};

    function set(key, value) {
      ctx.set(key, value);
      headersSet[key] = value;
    }

   // 添加响应头
    if (ctx.method !== 'OPTIONS') {
      // Simple Cross-Origin Request, Actual Request, and Redirects
      set('Access-Control-Allow-Origin', origin);

      if (credentials === true) {
        set('Access-Control-Allow-Credentials', 'true');
      }

      if (options.exposeHeaders) {
        set('Access-Control-Expose-Headers', options.exposeHeaders);
      }

      if (options.secureContext) {
        set('Cross-Origin-Opener-Policy', 'same-origin');
        set('Cross-Origin-Embedder-Policy', 'require-corp');
      }

      if (!options.keepHeadersOnError) {
        return await next();
      }
      try {
        return await next();
      } catch (err) {
        const errHeadersSet = err.headers || {};
        const varyWithOrigin = vary.append(errHeadersSet.vary || errHeadersSet.Vary || '', 'Origin');
        delete errHeadersSet.Vary;

        err.headers = {
          ...errHeadersSet,
          ...headersSet,
          ...{ vary: varyWithOrigin },
        };
        throw err;
      }
    } else {
      // Preflight Request

      // If there is no Access-Control-Request-Method header or if parsing failed,
      // do not set any additional headers and terminate this set of steps.
      // The request is outside the scope of this specification.
      if (!ctx.get('Access-Control-Request-Method')) {
        // this not preflight request, ignore it
        return await next();
      }

      ctx.set('Access-Control-Allow-Origin', origin);

      if (credentials === true) {
        ctx.set('Access-Control-Allow-Credentials', 'true');
      }

      if (options.maxAge) {
        ctx.set('Access-Control-Max-Age', options.maxAge);
      }

      if (options.privateNetworkAccess && ctx.get('Access-Control-Request-Private-Network')) {
        ctx.set('Access-Control-Allow-Private-Network', 'true');
      }

      if (options.allowMethods) {
        ctx.set('Access-Control-Allow-Methods', options.allowMethods);
      }

      if (options.secureContext) {
        set('Cross-Origin-Opener-Policy', 'same-origin');
        set('Cross-Origin-Embedder-Policy', 'require-corp');
      }

      let allowHeaders = options.allowHeaders;
      if (!allowHeaders) {
        allowHeaders = ctx.get('Access-Control-Request-Headers');
      }
      if (allowHeaders) {
        ctx.set('Access-Control-Allow-Headers', allowHeaders);
      }

      ctx.status = 204;
    }
  };
};

```

