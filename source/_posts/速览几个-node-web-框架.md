---
title: 速览几个 node web 框架
date: 2022-04-07 16:48:22
tags:
---

看了 koajs 的源码,意犹未尽,再看看几个其他的框架或者库.

<!-- more -->

# 简介

看了 koajs 的源码之后, 惊叹代码之少, 设计有趣. 看看其他几个框架. 看到了 fastify 有一个 benchmarks https://github.com/fastify/benchmarks, 那么就从这个 benchmarks 开始看吧.

从 npm 上看下载量如果下载量太少,说明没有人用, 就不看了.

# polkadot

[polkadot](https://github.com/lukeed/polkadot/)

    The tiny HTTP server that gets out of your way! ・

七天下载量不到 1000, 但是和 Polka 是同门师兄弟, 所以看看.

在上面的 benchmarks 中, polkadot 排名第一. 自我描述是一个 thiny HTTP server. 源代码也很精简.

```JavaScript
const { createServer } = require('http');
const { parse } = require('querystring');
const send = require('@polka/send-type');
const url = require('@polka/url');

function loop(res, out) {
	if (res.finished) return;
	return out && out.then !== void 0
		? out.then(d => loop(res, d)) : send(res, res.statusCode || 200, out);
}

module.exports = function (handler) {
	const $ = {
		server: null,

		handler(req, res) {
			const info = url(req);
			req.path = info.pathname;
			req.query = info.query ? parse(info.query) : {};
			req.search = info.search;

			return loop(res, handler(req, res));
		},

		listen() {
			const handler = (r1, r2) => setImmediate($.handler, r1, r2);
			($.server = $.server || createServer()).on('request', handler);
			return $.server.listen.apply($.server, arguments);
		}
	};

	return $;
}
```

短短几十行代码, 一个简单的封装, 并且实现了一个递归函数 loop 来处理 Promise. 不支持路由, 不支持中间件.

他这里使用 `setTmmediate` 看不出来有什么意义, (找到了作者解释 https://github.com/lukeed/polkadot/issues/5), 因为这里没有什么耗时操作, 或者会执行的东西.

# Polka

[Polka](https://github.com/lukeed/polka)

    A micro web server so fast, it'll make you dance! 👯

npm 七天下载量 8w 左右, 和上面 polkadot 是一个作者. 并且积极更新.

    Essentially, Polka is just a native HTTP server with added support for routing, middleware, and sub-applications. That's it! 🎉

从描述看 polka 实现了中间件和 router

```JavaScript
const polka = require('polka');

function one(req, res, next) {
  req.hello = 'world';
  next();
}

function two(req, res, next) {
  req.foo = '...needs better demo 😔';
  next();
}

polka()
  .use(one, two)
  .get('/users/:id', (req, res) => {
    console.log(`~> Hello, ${req.hello}`);
    res.end(`User: ${req.params.id}`);
  })
  .listen(3000, () => {
    console.log(`> Running on localhost:3000`);
  });
```

核心源代码也比较简单

```JavaScript

const http = require('http');
const Router = require('trouter');
const { parse } = require('querystring');
const parser = require('@polka/url');

function lead(x) {
	return x.charCodeAt(0) === 47 ? x : ('/' + x);
}

function value(x) {
  let y = x.indexOf('/', 1);
  return y > 1 ? x.substring(0, y) : x;
}

function mutate(str, req) {
	req.url = req.url.substring(str.length) || '/';
	req.path = req.path.substring(str.length) || '/';
}

function onError(err, req, res, next) {
	let code = (res.statusCode = err.code || err.status || 500);
	if (typeof err === 'string' || Buffer.isBuffer(err)) res.end(err);
	else res.end(err.message || http.STATUS_CODES[code]);
}

class Polka extends Router {
	constructor(opts={}) {
		super(opts);
		this.apps = {};
		this.wares = [];
		this.bwares = {};
		this.parse = parser;
		this.server = opts.server;
		this.handler = this.handler.bind(this);
		this.onError = opts.onError || onError; // catch-all handler
		this.onNoMatch = opts.onNoMatch || this.onError.bind(null, { code:404 });
	}

	add(method, pattern, ...fns) {
		let base = lead(value(pattern));
		if (this.apps[base] !== void 0) throw new Error(`Cannot mount ".${method.toLowerCase()}('${lead(pattern)}')" because a Polka application at ".use('${base}')" already exists! You should move this handler into your Polka application instead.`);
		return super.add(method, pattern, ...fns);
	}

	use(base, ...fns) {
		if (typeof base === 'function') {
			this.wares = this.wares.concat(base, fns);
		} else if (base === '/') {
			this.wares = this.wares.concat(fns);
		} else {
			base = lead(base);
			fns.forEach(fn => {
				if (fn instanceof Polka) {
					this.apps[base] = fn;
				} else {
					let arr = this.bwares[base] || [];
					arr.length > 0 || arr.push((r, _, nxt) => (mutate(base, r),nxt()));
					this.bwares[base] = arr.concat(fn);
				}
			});
		}
		return this; // chainable
	}

	listen() {
		(this.server = this.server || http.createServer()).on('request', this.handler);
		this.server.listen.apply(this.server, arguments);
		return this;
	}

	handler(req, res, info) {
		info = info || this.parse(req);
		let fns=[], arr=this.wares, obj=this.find(req.method, info.pathname);
		req.originalUrl = req.originalUrl || req.url;
		let base = value(req.path = info.pathname);
		if (this.bwares[base] !== void 0) {
			arr = arr.concat(this.bwares[base]);
		}
		if (obj) {
			fns = obj.handlers;
			req.params = obj.params;
		} else if (this.apps[base] !== void 0) {
			mutate(base, req); info.pathname=req.path; //=> updates
			fns.push(this.apps[base].handler.bind(null, req, res, info));
		}
		fns.push(this.onNoMatch);
		// Grab addl values from `info`
		req.search = info.search;
		req.query = parse(info.query);
		// Exit if only a single function
		let i=0, len=arr.length, num=fns.length;
		if (len === i && num === 1) return fns[0](req, res);
		// Otherwise loop thru all middlware
		let next = err => err ? this.onError(err, req, res, next) : loop();
		let loop = _ => res.finished || (i < len) && arr[i++](req, res, next);
		arr = arr.concat(fns);
		len += num;
		loop(); // init
	}
}
```

可以看到, router 部分是使用 trouter 这个包来解析和匹配 path, 中间件的话, 使用了递归来处理这些中间件, 代码不如 koajs 的优美.

# 0http

[0http](https://github.com/BackendStack21/0http)

    Cero friction HTTP requests router. The need for speed!

npm 下载量在 5000 左右

从 readme 的描述来看, 0http 也是使用 trouter 来解析和匹配 path 的, 但是他使用了 LRU (最近最少使用) 缓存 来缓存查询结果, 所以性能会好一些.

```JavaScript

const Trouter = require('trouter')
const next = require('./../next')
const LRU = require('lru-cache')
const { parse } = require('regexparam')
const queryparams = require('../utils/queryparams')

module.exports = (config = {}) => {
  if (config.defaultRoute === undefined) {
    config.defaultRoute = (req, res) => {
      res.statusCode = 404
      res.end()
    }
  }
  if (config.errorHandler === undefined) {
    config.errorHandler = (err, req, res) => {
      res.statusCode = 500
      res.end(err.message)
    }
  }
  if (config.cacheSize === undefined) {
    config.cacheSize = 1000
  }
  if (config.id === undefined) {
    config.id = (Date.now().toString(36) + Math.random().toString(36).substr(2, 5)).toUpperCase()
  }

  const routers = {}
  const isCacheEnabled = config.cacheSize > 0
  const cache = isCacheEnabled ? new LRU(config.cacheSize) : null
  const router = new Trouter()
  router.id = config.id

  const _use = router.use

  router.use = (prefix, ...middlewares) => {
    if (typeof prefix === 'function') {
      middlewares = [prefix]
      prefix = '/'
    }
    _use.call(router, prefix, middlewares)

    if (middlewares[0].id) {
      // caching router -> pattern relation for urls pattern replacement
      const { pattern } = parse(prefix, true)
      routers[middlewares[0].id] = pattern
    }

    return this
  }

  router.lookup = (req, res, step) => {
    if (!req.url) {
      req.url = '/'
    }
    if (!req.originalUrl) {
      req.originalUrl = req.url
    }

    queryparams(req, req.url)

    let match
    if (isCacheEnabled) {
      const reqCacheKey = req.method + req.path
      match = cache.get(reqCacheKey)
      if (!match) {
        match = router.find(req.method, req.path)
        cache.set(reqCacheKey, match)
      }
    } else {
      match = router.find(req.method, req.path)
    }

    if (match.handlers.length !== 0) {
      const middlewares = [...match.handlers]
      if (step !== undefined) {
        // router is being used as a nested router
        middlewares.push((req, res, next) => {
          req.url = req.preRouterUrl
          req.path = req.preRouterPath

          delete req.preRouterUrl
          delete req.preRouterPath

          return step()
        })
      }

      if (!req.params) {
        req.params = {}
      }
      Object.assign(req.params, match.params)

      return next(middlewares, req, res, 0, routers, config.defaultRoute, config.errorHandler)
    } else {
      config.defaultRoute(req, res)
    }
  }

  router.on = (method, pattern, ...handlers) => router.add(method, pattern, handlers)

  return router
}
```

核心就是 router 这部分代码. 和前面看的差不太多, 做了一个优化处理, 挺好的.

