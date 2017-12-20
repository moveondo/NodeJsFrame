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

