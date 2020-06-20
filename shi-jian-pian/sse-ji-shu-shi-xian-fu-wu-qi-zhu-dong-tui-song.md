# SSE 技术实现服务器主动推送

### 开始快速

SSE 技术就是实现了 服务端 向客户端主动推送数据，而不是依赖

```javascript

var http = require("http");

http.createServer(function (req, res) {
  if (fileName === "/stream") {
    res.writeHead(200, {
      "Content-Type":"text/event-stream",
      "Access-Control-Allow-Origin": '*',
    });
    
    interval = setInterval(function () {
      // res.write('event: test\n'); // 事件类型
      // res.write('event: message\n'); // 如果不指定事件，这里默认是 message
      res.write("data: " + (new Date()) + "\n\n"); 
    }, 1000);

    req.connection.addListener("close", function () {
      clearInterval(interval);
    }, false);
  }
}).listen(8844, "127.0.0.1");
```



```javascript
router.get('/getrand_sse', async (ctx, next) => {
  ctx.set({
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });
  ctx.status = 200;
  const fstream = new stream.PassThrough()
  ctx.body = fstream; 
  let counter = 5;
  const t = setInterval(() => {
    fstream.write(`data: ${JSON.stringify(rand.randword_text())}\n\n`);
    counter --;
    if (counter === 0) {
      console.log('end')
      fstream.write('event: pause\n\n');
      fstream.end();
      clearInterval(t);
    }
  }, 5200);
});
```

