# Cookie的使用 登录与验证

## Cookie的使用 登录与验证 <a id="cookie-de-shi-yong-deng-lu-yu-yan-zheng"></a>

‌

cookie是什么？这里简单的讲，就是登录状态，比如你在网站上登录了账号，只需要执行一个登录操作，后面的操作我都知道是你，这个是怎么做到或者实现的呢？ 这里使用的就是 cookie技术。‌

可以简单的理解，你登录之后，服务端给你一个密码，来鉴别，你是不是已经登录的那位。‌

Cookie 本身的存储是十分简单的，是使用键值对的方式来进行存储的，也就是 `'key':' value'的这种形式。`‌

### 快速开始 <a id="kuai-su-kai-shi"></a>

‌

这里通过简简单单的例子来模拟一个Cookie的使用方式。‌

下面的这个函数是用来检查 Cookie 的，代码不难看懂，就是来得到cookie里面的一个值，如果有name的值就正常，否则就不认识你。这里的Cookie 怎么得到呢？ 这里就使用 `cookie.set`的方法来设置cookie的值。

```text
router.get('/chk_cookie', async (ctx, next) => {  if (ctx.cookies.get('name') == undefined) {    ctx.body = "i don't know you!"  } else {    ctx.body = 'hey, there ' + ctx.cookies.get('name')  }});
```

‌

设置cookie的代码在下面，很容易的实现了一个 设置 cookie的功能。

```text
router.get('/get_cookie', async (ctx, next) => {  ctx.cookies.set(    'name',     'ann'  )  ctx.body = 'cookie is ok'});
```

‌

### 简单的登录 <a id="jian-dan-de-deng-lu"></a>

‌

前面知道了cookie是用来记载登录状态的，后面就来使用它，来完成一次登陆，以及登录的验证。‌

这里是我们的首页，首先判断name有没有，如果没有说明没有登录，就把页面重定向到 login。

```text
router.get('/', async (ctx, next) => {  if (ctx.cookies.get('name') == undefined) {    ctx.redirect('/login')  } else {    ctx.body = 'hey, there ' + ctx.cookies.get('name')  }});
```

‌

关于login的函数，由于目前没有前端，只能使用 get的方式来简简单单搞一下，下面加个例子，看起来害怕，但是实际上的核心代码就是那一个if语句，当然在实际上没有这么简单，但是基本的原理是一样的。

```text
router.get('/login', async (ctx, next) => {    loginInfo = ctx.request.query    console.log(loginInfo.username)    console.log(loginInfo.password)    if (loginInfo.username === 'ann' && loginInfo.password === '123') {        ctx.cookies.set(            'name',             'ann',          )          ctx.body = 'login in success'     } else {        ctx.status = 401        ctx.body = 'login in fail'      }});

```

