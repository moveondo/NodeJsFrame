### cookie

cookie的读取和设置。

```
this.cookies.get('view');
this.cookies.set('view', n);

```
get和set方法都可以接受第三个参数，表示配置参数。其中的signed参数，用于指定cookie是否加密。如果指定加密的话，必须用app.keys指定加密短语。

> app.keys = ['secret1', 'secret2'];

> this.cookies.set('name', '张三', { signed: true });

this.cookie的配置对象的属性如下。

>signed：cookie是否加密。

>expires：cookie何时过期

>path：cookie的路径，默认是“/”。

>domain：cookie的域名。

>secure：cookie是否只有https请求下才发送。

>httpOnly：是否只有服务器可以取到cookie，默认为true

### session

```
var session = require('koa-session');
var koa = require('koa');
var app = koa();

app.keys = ['some secret hurr'];
app.use(session(app));

app.use(function *(){
  var n = this.session.views || 0;
  this.session.views = ++n;
  this.body = n + ' views';
})

app.listen(3000);
console.log('listening on port 3000');

```
