### Request对象

Request对象表示HTTP请求。

（1）this.request.header

返回一个对象，包含所有HTTP请求的头信息。它也可以写成this.request.headers。

（2）this.request.method

返回HTTP请求的方法，该属性可读写。

（3）this.request.length

返回HTTP请求的Content-Length属性，取不到值，则返回undefined。

（4）this.request.path

返回HTTP请求的路径，该属性可读写。

（5）this.request.href

返回HTTP请求的完整路径，包括协议、端口和url。

```
this.request.href
// http://example.com/foo/bar?q=1

```
（6）this.request.querystring

返回HTTP请求的查询字符串，不含问号。该属性可读写。

（7）this.request.search

返回HTTP请求的查询字符串，含问号。该属性可读写。

（8）this.request.host

返回HTTP请求的主机（含端口号）。

（9）this.request.hostname

返回HTTP的主机名（不含端口号）。

（10）this.request.type

返回HTTP请求的Content-Type属性。

```
var ct = this.request.type;
// "image/png"

```
（11）this.request.charset

返回HTTP请求的字符集。
```
this.request.charset
// "utf-8"
```

（12）this.request.query

返回一个对象，包含了HTTP请求的查询字符串。如果没有查询字符串，则返回一个空对象。该属性可读写。

比如，查询字符串color=blue&size=small，会得到以下的对象。
```
{
  color: 'blue',
  size: 'small'
}
```
（13）this.request.fresh

返回一个布尔值，表示缓存是否代表了最新内容。通常与If-None-Match、ETag、If-Modified-Since、Last-Modified等缓存头，配合使用。

```
this.response.set('ETag', '123');

// 检查客户端请求的内容是否有变化
if (this.request.fresh) {
  this.response.status = 304;
  return;
}

// 否则就表示客户端的内容陈旧了，
// 需要取出新内容
this.response.body = yield db.find('something');

```
（14）this.request.stale

返回this.request.fresh的相反值。

（15）this.request.protocol

返回HTTP请求的协议，https或者http。

（16）this.request.secure

返回一个布尔值，表示当前协议是否为https。

（17）this.request.ip

返回发出HTTP请求的IP地址。

（18）this.request.subdomains

返回一个数组，表示HTTP请求的子域名。该属性必须与app.subdomainOffset属性搭配使用。app.subdomainOffset属性默认为2，则域名“tobi.ferrets.example.com”返回[“ferrets”, “tobi”]，如果app.subdomainOffset设为3，则返回[“tobi”]。

（19）this.request.is(types…)

返回指定的类型字符串，表示HTTP请求的Content-Type属性是否为指定类型。

```
// Content-Type为 text/html; charset=utf-8
this.request.is('html'); // 'html'
this.request.is('text/html'); // 'text/html'
this.request.is('text/*', 'text/html'); // 'text/html'

// Content-Type为 application/json
this.request.is('json', 'urlencoded'); // 'json'
this.request.is('application/json'); // 'application/json'
this.request.is('html', 'application/*'); // 'application/json'

```
如果不满足条件，返回false；如果HTTP请求不含数据，则返回undefined。
```
this.is('html'); // false
```
它可以用于过滤HTTP请求，比如只允许请求下载图片。
```
if (this.is('image/*')) {
  // process
} else {
  this.throw(415, 'images only!');
}
```
（20）this.request.accepts(types)

检查HTTP请求的Accept属性是否可接受，如果可接受，则返回指定的媒体类型，否则返回false。
```
// Accept: text/html
this.request.accepts('html');
// "html"

// Accept: text/*, application/json
this.request.accepts('html');
// "html"
this.request.accepts('text/html');
// "text/html"
this.request.accepts('json', 'text');
// => "json"
this.request.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
this.request.accepts('image/png');
this.request.accepts('png');
// false

// Accept: text/*;q=.5, application/json
this.request.accepts(['html', 'json']);
this.request.accepts('html', 'json');
// "json"

// No Accept header
this.request.accepts('html', 'json');
// "html"
this.request.accepts('json', 'html');
// => "json"

```
如果accepts方法没有参数，则返回所有支持的类型（text/html,application/xhtml+xml,image/webp,application/xml,/）。

如果accepts方法的参数有多个参数，则返回最佳匹配。如果都不匹配则返回false，并向客户端抛出一个406”Not Acceptable“错误。

如果HTTP请求没有Accept字段，那么accepts方法返回它的第一个参数。

accepts方法可以根据不同Accept字段，向客户端返回不同的字段。

```
switch (this.request.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: this.throw(406, 'json, html, or text only');
}

```
（21）this.request.acceptsEncodings(encodings)

该方法根据HTTP请求的Accept-Encoding字段，返回最佳匹配，如果没有合适的匹配，则返回false。
```
// Accept-Encoding: gzip
this.request.acceptsEncodings('gzip', 'deflate', 'identity');
// "gzip"
this.request.acceptsEncodings(['gzip', 'deflate', 'identity']);
// "gzip"
```
注意，acceptEncodings方法的参数必须包括identity（意为不编码）。

如果HTTP请求没有Accept-Encoding字段，acceptEncodings方法返回所有可以提供的编码方法。
```
// Accept-Encoding: gzip, deflate
this.request.acceptsEncodings();
// ["gzip", "deflate", "identity"]
```
（22）this.request.acceptsCharsets(charsets)

该方法根据HTTP请求的Accept-Charset字段，返回最佳匹配，如果没有合适的匹配，则返回false。
```
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
this.request.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

this.request.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```
如果acceptsCharsets方法没有参数，则返回所有可接受的匹配。
```
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
this.request.acceptsCharsets();
// ["utf-8", "utf-7", "iso-8859-1"]
```

如果都不匹配，acceptsCharsets方法返回false，并向客户端抛出一个406“Not Acceptable”错误。

如果都不匹配，acceptsEncodings方法返回false，并向客户端抛出一个406“Not Acceptable”错误。

（23）this.request.acceptsLanguages(langs)

该方法根据HTTP请求的Accept-Language字段，返回最佳匹配，如果没有合适的匹配，则返回false。
```
// Accept-Language: en;q=0.8, es, pt
this.request.acceptsLanguages('es', 'en');
// "es"
this.request.acceptsLanguages(['en', 'es']);
// "es"
如果acceptsCharsets方法没有参数，则返回所有可接受的匹配。

// Accept-Language: en;q=0.8, es, pt
this.request.acceptsLanguages();
// ["es", "pt", "en"]
```
如果都不匹配，acceptsLanguages方法返回false，并向客户端抛出一个406“Not Acceptable”错误。

（24）this.request.socket

返回HTTP请求的socket。

（25）this.request.get(field)

返回HTTP请求指定的字段。

## Response对象

Response对象表示HTTP回应。

（1）this.response.header

返回HTTP回应的头信息。

（2）this.response.socket

返回HTTP回应的socket。

（3）this.response.status

返回HTTP回应的状态码。默认情况下，该属性没有值。该属性可读写，设置时等于一个整数。

（4）this.response.message

返回HTTP回应的状态信息。该属性与this.response.message是配对的。该属性可读写。

（5）this.response.length

返回HTTP回应的Content-Length字段。该属性可读写，如果没有设置它的值，koa会自动从this.request.body推断。

（6）this.response.body

返回HTTP回应的信息体。该属性可读写，它的值可能有以下几种类型。

> 字符串：Content-Type字段默认为text/html或text/plain，字符集默认为utf-8，Content-Length字段同时设定。

> 二进制Buffer：Content-Type字段默认为application/octet-stream，Content-Length字段同时设定。

> Stream：Content-Type字段默认为application/octet-stream。

> JSON对象：Content-Type字段默认为application/json。

> null（表示没有信息体）

如果this.response.status没设置，Koa会自动将其设为200或204。

（7）this.response.get(field)

返回HTTP回应的指定字段。

> var etag = this.get('ETag');

注意，get方法的参数是区分大小写的。

（8）this.response.set()

设置HTTP回应的指定字段。

> this.set('Cache-Control', 'no-cache');

set方法也可以接受一个对象作为参数，同时为多个字段指定值。
```
this.set({
  'Etag': '1234',
  'Last-Modified': date
});
```
（9）this.response.remove(field)

移除HTTP回应的指定字段。

（10）this.response.type

返回HTTP回应的Content-Type字段，不包括“charset”参数的部分。
```
var ct = this.reponse.type;
// "image/png"
该属性是可写的。

this.reponse.type = 'text/plain; charset=utf-8';
this.reponse.type = 'image/png';
this.reponse.type = '.png';
this.reponse.type = 'png';
```

设置type属性的时候，如果没有提供charset参数，Koa会判断是否自动设置。如果this.response.type设为html，charset默认设为utf-8；但如果this.response.type设为text/html，就不会提供charset的默认值。

（10）this.response.is(types…)

该方法类似于this.request.is()，用于检查HTTP回应的类型是否为支持的类型。

它可以在中间件中起到处理不同格式内容的作用。
```
var minify = require('html-minifier');

app.use(function *minifyHTML(next){
  yield next;

  if (!this.response.is('html')) return;

  var body = this.response.body;
  if (!body || body.pipe) return;

  if (Buffer.isBuffer(body)) body = body.toString();
  this.response.body = minify(body);
});
```
上面代码是一个中间件，如果输出的内容类型为HTML，就会进行最小化处理。

（11）this.response.redirect(url, [alt])

该方法执行302跳转到指定网址。
```
this.redirect('back');
this.redirect('back', '/index.html');
this.redirect('/login');
this.redirect('http://google.com');
```
如果redirect方法的第一个参数是back，将重定向到HTTP请求的Referrer字段指定的网址，如果没有该字段，则重定向到第二个参数或“/”网址。

如果想修改302状态码，或者修改body文字，可以采用下面的写法。
```
this.status = 301;
this.redirect('/cart');
this.body = 'Redirecting to shopping cart';
```

（12）this.response.attachment([filename])

该方法将HTTP回应的Content-Disposition字段，设为“attachment”，提示浏览器下载指定文件。

（13）this.response.headerSent

该方法返回一个布尔值，检查是否HTTP回应已经发出。

（14）this.response.lastModified

该属性以Date对象的形式，返回HTTP回应的Last-Modified字段（如果该字段存在）。该属性可写。

> this.response.lastModified = new Date();

（15）this.response.etag

该属性设置HTTP回应的ETag字段。

> this.response.etag = crypto.createHash('md5').update(this.body).digest('hex');

注意，不能用该属性读取ETag字段。

（16）this.response.vary(field)

该方法将参数添加到HTTP回应的Vary字段。
