### CSRF攻击

CSRF攻击是指用户的session被劫持，用来冒充用户的攻击。

koa-csrf插件用来防止CSRF攻击。原理是在session之中写入一个秘密的token，用户每次使用POST方法提交数据的时候，必须含有这个token，否则就会抛出错误。
```
var koa = require('koa');
var session = require('koa-session');
var csrf = require('koa-csrf');
var route = require('koa-route');

var app = module.exports = koa();

app.keys = ['session key', 'csrf example'];
app.use(session(app));

app.use(csrf());

app.use(route.get('/token', token));
app.use(route.post('/post', post));

function* token () {
  this.body = this.csrf;
}

function* post() {
  this.body = {ok: true};
}

app.listen(3000);
```

POST请求含有token，可以是以下几种方式之一，koa-csrf插件就能获得token。

>表单的_csrf字段

>查询字符串的_csrf字段

>HTTP请求头信息的x-csrf-token字段

>HTTP请求头信息的x-xsrf-token字段

### 数据压缩

koa-compress模块可以实现数据压缩。
```
app.use(require('koa-compress')())
app.use(function* () {
  this.type = 'text/plain'
  this.body = fs.createReadStream('filename.txt')
})
```
### 源码解读

每一个网站就是一个app，它由lib/application定义。
```
function Application() {
  if (!(this instanceof Application)) return new Application;
  this.env = process.env.NODE_ENV || 'development';
  this.subdomainOffset = 2;
  this.middleware = [];
  this.context = Object.create(context);
  this.request = Object.create(request);
  this.response = Object.create(response);
}

var app = Application.prototype;

exports = module.exports = Application;
```
app.use()用于注册中间件，即将Generator函数放入中间件数组。
```
app.use = function(fn){
  if (!this.experimental) {
    // es7 async functions are allowed
    assert(fn && 'GeneratorFunction' == fn.constructor.name, 'app.use() requires a generator function');
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  return this;
};
```
app.listen()就是http.createServer(app.callback()).listen(...)的缩写。
```
app.listen = function(){
  debug('listen');
  var server = http.createServer(this.callback());
  return server.listen.apply(server, arguments);
};

app.callback = function(){
  var mw = [respond].concat(this.middleware);
  var fn = this.experimental
    ? compose_es7(mw)
    : co.wrap(compose(mw));
  var self = this;

  if (!this.listeners('error').length) this.on('error', this.onerror);

  return function(req, res){
    res.statusCode = 404;
    var ctx = self.createContext(req, res);
    onFinished(res, ctx.onerror);
    fn.call(ctx).catch(ctx.onerror);
  }
};
```

上面代码中，app.callback()会返回一个函数，用来处理HTTP请求。它的第一行mw = [respond].concat(this.middleware)，表示将respond函数（这也是一个Generator函数）放入this.middleware，现在mw就变成了[respond, S1, S2, S3]。

compose(mw)将中间件数组转为一个层层调用的Generator函数。
```
function compose(middleware){
  return function *(next){
    if (!next) next = noop();

    var i = middleware.length;

    while (i--) {
      next = middleware[i].call(this, next);
    }

    yield *next;
  }
}

function *noop(){}
```

上面代码中，下一个generator函数总是上一个Generator函数的参数，从而保证了层层调用。

var fn = co.wrap(gen)则是将Generator函数包装成一个自动执行的函数，并且返回一个Promise。

```
//co package
co.wrap = function (fn) {
  return function () {
    return co.call(this, fn.apply(this, arguments));
  };
};

```
由于co.wrap(compose(mw))执行后，返回的是一个Promise，所以可以对其使用catch方法指定捕捉错误的回调函数fn.call(ctx).catch(ctx.onerror)。

将所有的上下文变量都放进context对象。
```
app.createContext = function(req, res){
  var context = Object.create(this.context);
  var request = context.request = Object.create(this.request);
  var response = context.response = Object.create(this.response);
  context.app = request.app = response.app = this;
  context.req = request.req = response.req = req;
  context.res = request.res = response.res = res;
  request.ctx = response.ctx = context;
  request.response = response;
  response.request = request;
  context.onerror = context.onerror.bind(context);
  context.originalUrl = request.originalUrl = req.url;
  context.cookies = new Cookies(req, res, this.keys);
  context.accept = request.accept = accepts(req);
  context.state = {};
  return context;
};
```

真正处理HTTP请求的是下面这个Generator函数。

```
function *respond(next) {
  yield *next;

  // allow bypassing koa
  if (false === this.respond) return;

  var res = this.res;
  if (res.headersSent || !this.writable) return;

  var body = this.body;
  var code = this.status;

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    this.body = null;
    return res.end();
  }

  if ('HEAD' == this.method) {
    if (isJSON(body)) this.length = Buffer.byteLength(JSON.stringify(body));
    return res.end();
  }

  // status body
  if (null == body) {
    this.type = 'text';
    body = this.message || String(code);
    this.length = Buffer.byteLength(body);
    return res.end(body);
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res);

  // body: json
  body = JSON.stringify(body);
  this.length = Buffer.byteLength(body);
  res.end(body);
}
```






