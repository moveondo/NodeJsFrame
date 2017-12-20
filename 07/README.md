### 路由

可以通过this.path属性，判断用户请求的路径，从而起到路由作用。
```
app.use(function* (next) {
  if (this.path === '/') {
    this.body = 'we are at home!';
  } else {
    yield next;
  }
})

// 等同于

app.use(function* (next) {
  if (this.path !== '/') return yield next;
  this.body = 'we are at home!';
})

```
下面是多路径的例子。

```
let koa = require('koa')

let app = koa()

// normal route
app.use(function* (next) {
  if (this.path !== '/') {
    return yield next
  }

  this.body = 'hello world'
});

// /404 route
app.use(function* (next) {
  if (this.path !== '/404') {
    return yield next;
  }

  this.body = 'page not found'
});

// /500 route
app.use(function* (next) {
  if (this.path !== '/500') {
    return yield next;
  }

  this.body = 'internal server error'
});

app.listen(8080)
```
上面代码中，每一个中间件负责一个路径，如果路径不符合，就传递给下一个中间件。

复杂的路由需要安装koa-router插件。

```
var app = require('koa')();
var Router = require('koa-router');

var myRouter = new Router();

myRouter.get('/', function *(next) {
  this.response.body = 'Hello World!';
});

app.use(myRouter.routes());

app.listen(3000);

```
上面代码对根路径设置路由。

Koa-router实例提供一系列动词方法，即一种HTTP动词对应一种方法。典型的动词方法有以下五种。

> router.get()  
> router.post()  
> router.put()  
> router.del()  
> router.patch()  
