# 如何自己写一个中间件

### 中间件是个啥

有学习到KOA之后对中间件这个概念算是接触了好多好多次了，这里来对中间件是个啥来进行总结

总结一下，中间件就是在处理请求和构造响应之间执行的东西（权且可以理解为一个函数），这个函数将会在我们server对处理的请求中来进行自动的执行，来得到我们的想要的功能。比如：

* KoaBody 执行过程中自动的来将POST的body存放到ctx中
* KoaRouter 帮我们来设置路由路径与后面的服务（接口）函数的对应关系
* KoaStatic 来自动的把我们的访问的路径在配置的目录中来进行查找

 综上，可以看出来，中间件就是KOA框架里面的模块，避免了重复的代码来实现基础的功能，所以说，代码是越写越简单的。

### 中间件是怎么运行的

KOA的官网上的介绍，中间件是按照链式的方式来执行的。所以这里来看一个使用的例子，下面的几行代码就是KOA中中间件的一个使用的实例，这里就引入了两个中间件，一个啥都不做，一个做点什么。

```javascript
const Koa = require('koa');
const app = new Koa();

app.use(async function(ctx, next) {
  // logger Call
  await next();
  // logger return
  // console.log(ctx.request.query);
});

app.use(async function(ctx, next) {
    await next();
    const rt = ctx.response.get('X-Response-Time');
    console.log(`${ctx.method} ${ctx.url} - ${rt}`);
});
```

根据use的顺序，首先会来执行第一个中间件的函数，看到传入的参数有 ctx 和 next，这里的ctx 就是这一次http请求的context上下文，next 这个是比较重要的东西，实际上就是下一个中间件的对应的函数，进行await调用，

之后就到了第二个函数，函数中首先await调用 next， 但实际上没有下一个中间件了，所以这里的next 会执行一个空函数。而后空函数返回，继续执行下面的response 的 get header 的函数。

至此，两个中间件的工作就完成了。总结一下就是链式调用，使用 `await next` 来实现到下一个中间件的调用。

### 自己写一个

有了上中间件的基础，就考虑怎么开始自己写一个，来写一个类似于 Koabody的中间件，最基本的作用就是来帮我们把POST请求中的body给提取出来

#### 如何提取参数

既然这个中间件的功能是来提取 post请求的 body内容，我们就需要先从需求本身出发，先实现用来提取post内容的代码。这里就当我是刚刚写的（实际上是复制粘贴），下面的代码可以看到实际上就是http模块里面的 data 和 end 的两个事件，用来处理post 的分块传输以及传输完成的事件。

```javascript
  let parsedData = await new Promise((resolve, reject) => {
    try {
      let postdata = "";
      ctx.req.addListener('data', (data) => {
        postdata += data
      })
      ctx.req.addListener("end", function () {
        // // console.log(parseData);
        resolve(postdata);
      })
    } catch (err) {
      reject(err)
    }
  })
```

#### 变成中间件

上面已经写好了 POST体的提取的代码，现在的步骤就是把代码做成中间件的形式，这样就不需要在每一个 router用到postbody的时候再重复使用上面的那一堆的代码了。怎么做呢，及的ctx这个东西吗？在KOA的请求中，他是一直存在的，只要我们把结果存在这个对象里面，就可以随时随地的来拿到这个结果。

最后的代码看起来是这个样子的，把提取到的值直接赋值给ctx中的一个变量。这样只要有post的请求这里的parsedata就是我们的postbody

```javascript
app.use( async (ctx, next) => {
  ctx.parsedData = await new Promise((resolve, reject) => {
    try {
      let postdata = "";
      ctx.req.addListener('data', (data) => {
        postdata += data
      })
      ctx.req.addListener("end", function () {
        resolve(postdata);
      })
    } catch (err) {
      reject(err)
    }
  })
}
```

有了这个中间件之后，我们就可以直接在上传代码里面这样写

```javascript
router.post('/users0', async (ctx) => {
  parsedData = ctx.parsedData;
  let result = await new Promise ((resolve, reject) => {
  fs.writeFile(`./static/upload/${Math.random().toString()}.ann`, parsedData, 
    function(err){
      if(err) {
        reject(err)
      } else {
        resolve('写文件操作成功');
      } 
    })
  })
  ctx.body = result;  
});
```

至此，一个自制的中间件写完了！

