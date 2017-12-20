### context对象

中间件当中的this表示上下文对象context，代表一次HTTP请求和回应，即一次访问/回应的所有信息，都可以从上下文对象获得。context对象封装了request和response对象，并且提供了一些辅助方法。每次HTTP请求，就会创建一个新的context对象。

```
app.use(function *(){
  this; // is the Context
  this.request; // is a koa Request
  this.response; // is a koa Response
});

```

context对象的很多方法，其实是定义在ctx.request对象或ctx.response对象上面，比如:

ctx.type和ctx.length对应于ctx.response.type和ctx.response.length;

ctx.path和ctx.method对应于ctx.request.path和ctx.request.method。

#### context对象的全局属性。

> request：指向Request对象  

> response：指向Response对象

> req：指向Node的request对象

> res：指向Node的response对象

> app：指向App对象

> state：用于在中间件传递信息。

```
this.state.user = yield User.find(id);
```
上面代码中，user属性存放在this.state对象上面，可以被另一个中间件读取。

context对象的全局方法。

> throw()：抛出错误，直接决定了HTTP回应的状态码。

> assert()：如果一个表达式为false，则抛出一个错误。

```
this.throw(403);
this.throw('name required', 400);
this.throw('something exploded');

this.throw(400, 'name required');
// 等同于
var err = new Error('name required');
err.status = 400;
throw err;

```
assert方法的例子。

```
// 格式
ctx.assert(value, [msg], [status], [properties])

// 例子
this.assert(this.user, 401, 'User not found. Please login!');

```
以下模块解析POST请求的数据。

* co-body
* https://github.com/koajs/body-parser
* https://github.com/koajs/body-parsers
```
var parse = require('co-body');

// in Koa handler
var body = yield parse(this);
```

### 错误处理机制

Koa提供内置的错误处理机制，任何中间件抛出的错误都会被捕捉到，引发向客户端返回一个500错误，而不会导致进程停止，因此也就不需要forever这样的模块重启进程

```
app.use(function *() {
  throw new Error();
});

```
上面代码中，中间件内部抛出一个错误，并不会导致Koa应用挂掉。Koa内置的错误处理机制，会捕捉到这个错误。

当然，也可以额外部署自己的错误处理机制
```
app.use(function *() {
  try {
    yield saveResults();
  } catch (err) {
    this.throw(400, '数据无效');
  }
});

```
上面代码自行部署了try…catch代码块，一旦产生错误，就用this.throw方法抛出。该方法可以将指定的状态码和错误信息，返回给客户端。

对于未捕获错误，可以设置error事件的监听函数。

```
app.on('error', function(err){
  log.error('server error', err);
});
```
error事件的监听函数还可以接受上下文对象，作为第二个参数。

```
app.on('error', function(err, ctx){
  log.error('server error', err, ctx);
});

```
如果一个错误没有被捕获，koa会向客户端返回一个500错误“Internal Server Error”。

this.throw方法用于向客户端抛出一个错误。

```
this.throw(403);
this.throw('name required', 400);
this.throw(400, 'name required');
this.throw('something exploded');

this.throw('name required', 400)
// 等同于
var err = new Error('name required');
err.status = 400;
throw err;
```

this.throw方法的两个参数，一个是错误码，另一个是报错信息。如果省略状态码，默认是500错误。

this.assert方法用于在中间件之中断言，用法类似于Node的assert模块。

> this.assert(this.user, 401, 'User not found. Please login!');

上面代码中，如果this.user属性不存在，会抛出一个401错误。

由于中间件是层级式调用，所以可以把try { yield next }当成第一个中间件。
```
app.use(function *(next) {
  try {
    yield next;
  } catch (err) {
    this.status = err.status || 500;
    this.body = err.message;
    this.app.emit('error', err, this);
  }
});

app.use(function *(next) {
  throw new Error('some error');
})
```
