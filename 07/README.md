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

这些动词方法可以接受两个参数，第一个是路径模式，第二个是对应的控制器方法（中间件），定义用户请求该路径时服务器行为。

```
router.get('/', function *(next) {
  this.body = 'Hello World!';
});
```
上面代码中，router.get方法的第一个参数是根路径，第二个参数是对应的函数方法。

注意，路径匹配的时候，不会把查询字符串考虑在内。比如，/index?param=xyz匹配路径/index。

有些路径模式比较复杂，Koa-router允许为路径模式起别名。起名时，别名要添加为动词方法的第一个参数，这时动词方法变成接受三个参数。

```
router.get('user', '/users/:id', function *(next) {
 // ...
});

```
上面代码中，路径模式\users\:id的名字就是user。路径的名称，可以用来引用对应的具体路径，比如url方法可以根据路径名称，结合给定的参数，生成具体的路径。

```
router.url('user', 3);
// => "/users/3"

router.url('user', { id: 3 });
// => "/users/3"

```

上面代码中，user就是路径模式的名称，对应具体路径/users/:id。url方法的第二个参数3，表示给定id的值是3，因此最后生成的路径是/users/3。

Koa-router允许为路径统一添加前缀。

```
var router = new Router({
  prefix: '/users'
});

router.get('/', ...); // 等同于"/users"
router.get('/:id', ...); // 等同于"/users/:id"
```
路径的参数通过this.params属性获取，该属性返回一个对象，所有路径参数都是该对象的成员。

```
// 访问 /programming/how-to-node
router.get('/:category/:title', function *(next) {
  console.log(this.params);
  // => { category: 'programming', title: 'how-to-node' }
});
```
param方法可以针对命名参数，设置验证条件。

```
router
  .get('/users/:user', function *(next) {
    this.body = this.user;
  })
  .param('user', function *(id, next) {
    var users = [ '0号用户', '1号用户', '2号用户'];
    this.user = users[id];
    if (!this.user) return this.status = 404;
    yield next;
  })
  
```
上面代码中，如果/users/:user的参数user对应的不是有效用户（比如访问/users/3），param方法注册的中间件会查到，就会返回404错误。

redirect方法会将某个路径的请求，重定向到另一个路径，并返回301状态码
```
router.redirect('/login', 'sign-in');

// 等同于
router.all('/login', function *() {
  this.redirect('/sign-in');
  this.status = 301;
});
```
redirect方法的第一个参数是请求来源，第二个参数是目的地，两者都可以用路径模式的别名代替。

