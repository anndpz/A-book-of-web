---
description: Happy To Talk Pz
---

# HTTP

## [HTTP\(Happy To Talk Pz\) module](https://nodejs.org/zh-cn/docs/guides/anatomy-of-an-http-transaction/)

既然我们是写的后端，那么就从http开始下手

这一片文章很好，现在也就是按照这个里面的内容来进行讲解

> [一次 HTTP 传输解析](https://nodejs.org/zh-cn/docs/guides/anatomy-of-an-http-transaction/)

### 快速开始

直接开始了，下面的代码可以来创建一个 http的server，实际上是一个 `EventEmitter`用来触发事件

```javascript
const http = require('http');
const server = http.createServer((request, response) => {
  // magic happens here!
});
```

既然上面说了是一个 emitter，所以就会触发不同的事件。使用 on 来对事件进行监听就得到了下面的代码

```javascript
server.on('request', (request, response) => {
  // the same kind of magic happens here!
});
```

当触发了 request事件的时候，后面的函数将会被调用，可以看到，参数有两个，熟悉的req 和 res

### res 和 req

正如前面提到的，res 和 req 分别是当前的响应内容以及请求内容。

而且，先有请求之后才会有响应，**所以我们写的代码的作用，实际上就是把请求变成响应的过程**

所以按照pz的说法，这里就两步：

1. 解析请求
2. 构造响应

#### 解析请求

请求的内容都保存在 req 里面，也就是request 里面。怎么得到里面的内容呢？使用**结构**来做，实例代码如下

```javascript
const { method, url } = request;
const { headers } = request;
const userAgent = headers['user-agent'];
```

可以看到这里使用了 结构的方法来读取出来 req 里面的各种参数。

#### 构造响应

在解析了用户的请求之后，就开始构造我们想要的响应。来份返回给用户。

响应里面几个重要的东西，当然不是很全，但是实际上就是我们用到的东西

* 响应头    在你看我的请求之前对你的通知，比如我要用英语
* 返回码    你这个请求真实的状态 200 404 502
* 响应体    返回给你的东西

前面说到，我们得到了这个res，所以我们现在可以构建一个 响应出去

```javascript
response.setHeader('Content-Type', 'application/json');
response.setHeader('X-Powered-By', 'bacon');
response.statusCode = 404;
// 发送响应体
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
// 这里的end很重要，表示你已经整好了这个响应体了。
response.end();
```

### 如果是POST

如果是POST 的情况，就有另一个问题了，我们需要获得请求体。

正如你知道的 GET 的内容在URL里面比如 `www.aaa.com/bb?pz=xxx`，但是POST 的数据传递是在 body 中，而且重要的是，post 的数据传递可以很大，**比如上传图片视频都是使用的POST方法来进行，因为很大不能一次性传完**，所以就需要使用分块的方式来进行传输，具体的代码可以看下面，

```text
let boddy = [];
request.on('data', (chunk) => {
  boddy.push(chunk);
}).on('end', () => {
  body = Buffer.concat(body).toString();
  // at this point, `body` has the entire request body stored in it as a string
});
```

chunk 指的就是区块，在是数据区块的时候，就执行 `body.psuh`来把区块的内容给存起来，当发送了end 的时候，这里就直接把body给读出来。

* 思考使用get可以传输图片吗？怎么搞？

### 最简单的服务器

有了上面的铺垫，这里就实现一个最简单的服务器。怎么搞呢？

服务器的作用是什么？处理请求，构造响应对不？那我们开始，看下面给出的代码，不着急晕，来看解析

```javascript
const http = require('http');

http.createServer((request, response) => {
    const { headers, method, url } = request;
    let body = [];
    // 请求出现异常的回调
    request.on('error', (err) => {
      console.error(err);
    })
    // 请求是POST的回调韩式
    request.on('data', (chunk) => {
      body.push(chunk);
    })
    // ===========俺是分割线====================
    // 当请求完成，也就是请求被服务器完整的收到和处理，之后开始构造响应体
        body = Buffer.concat(body).toString();
      // 这里是处理响应时候的异常的回调
      response.on('error', (err) => {
        console.error(err);
      });

      // 开始构造响应内容，包括响应头，返回码，响应体
      response.statusCode = 200;
      response.setHeader('Content-Type', 'text/html');
      response.write("this is a book for Nodejs");
      response.end();
}).listen(8080);
```

### 中间件

中间件这个东西显得十分高大上的感觉，那么什么是中间件呢。**便于理解的来说，是请求到响应的中间处理的模块就是我们的中间件** 当然这个说法不准确或者错误。下面写一个最最最最简单的中间件。下面这个就是，我们把它用起来

```javascript
let simpleRoute = (path) => {
    routeMap = {
        'aaa': '1',
        'bbb': '2',
        'ccc': '3'
    }
    return routerMap[path]
}
```

写完了中间件我们来把它用上试试

```javascript
const http = require('http');

http.createServer((request, response) => {
    // 这里提取到 url
    const { headers, method, url } = request;
    request.on('end', () => {
        body = Buffer.concat(body).toString();
    });

    // 开始构造响应内容，包括响应头，返回码，响应体
    response.statusCode = 200;
    response.setHeader('Content-Type', 'text/html');
    // 使用中间件来实现 请求的url 到返回内容的映射
    let result = routerMap(map)
    response.write(result);
    response.end();
}).listen(8080);
```

* 思考这个中间件的作用是什么？

