---
description: Kiss Only Ann
---

# KOA

## [KOA\(Kiss Only Ann\)](https://koa.bootcss.com/)

如何来理解 KOA 这个东西呢，看一下官网的说明`Koa 是一个新的 web 框架`。

web框架是啥呢？实际上就是一个 http的模块，重点在框架两个字。框架是什么？就是一个架子，我们往里面填充想要的模块就可以了。比如熟悉的 `koa-router` 这个模块来看，他就是一个中间件，来对不同的urlpath来进行转发到不同的处理函数。要什么就装什么模块，来完成这个功能就不需要自己手写了。

### 快速开始

来使用框架快速实现一个 server，代码如下：

```javascript
const Koa = require('koa');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```

和上面的http是一样的，实现了一个 httpserver 的功能，实现了 **请求处理 以及 响应构造的两大功能** ，可能这里的过程不明显，但是 `helloworld` 就是我们返回的内容

### KOA 的中间件们

这里的代码比上面的更复杂，但是我们来慢慢分析。

```javascript
const Koa = require('koa');
const app = new Koa();

// logger
app.use(async (ctx, next) => {
  await next();
  const rt = ctx.response.get('X-Response-Time');
  console.log(`${ctx.method} ${ctx.url} - ${rt}`);
});

// x-response-time
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.set('X-Response-Time', `${ms}ms`);
});

// response
app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```

这里是 koa 官网给的一个实例的代码，来实现一个中间件的级联，哪里看出来集联呢，来看这个 `await next`的部分，这里看到是同步的执行了next这个函数，那么这里的next 是谁呢？就是这里的下一面的一个函数呀。

所以这里就出现了个调用返回链，从上到下执行，从下到上返回，有没有很美？

* 提问为什么在response里面没有 next 使用 next 可以吗？

  答可以，执行的将是一个空函数

### 常用的KOA中间件

* koa-router 路由中间件
* koa-static 静态文件中间件
* koa-views 模板渲染 
* koa-logger 访问日志打印

### http和KOA的结合

```javascript
const Koa = require('koa');
const app = new Koa();
app.listen(3000);
```

上面的代码是KOA里面创建一个server，之前有提到，实际上KOA只是web的框架，所以实际上就是http的包装，在官方的文档里面是这样写的，上面的代码等同于

```javascript
const http = require('http');
const Koa = require('koa');
const app = new Koa();
http.createServer(app.callback()).listen(3000);
```

所以就可以将一个koa实例绑定到多个端口/协议，或者多个到多个

```javascript
const http = require('http');
const https = require('https');
const Koa = require('koa');
const app = new Koa();
http.createServer(app.callback()).listen(3000);
https.createServer(app.callback()).listen(3001);
```

### KOAbody

