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
