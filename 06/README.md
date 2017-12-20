

Koa是一个类似于Express的Web开发框架，创始人也是同一个人。它的主要特点是，使用了ES6的Generator函数，进行了架构的重新设计。也就是说，Koa的原理和内部结构很像Express，但是语法和内部结构进行了升级。

官方faq有这样一个问题：”为什么koa不是Express 4.0？“，回答是这样的：”Koa与Express有很大差异，整个设计都是不同的，所以如果将Express 3.0按照这种写法升级到4.0，就意味着重写整个程序。所以，我们觉得创造一个新的库，是更合适的做法。“

## Koa应用

一个Koa应用就是一个对象，包含了一个middleware数组，这个数组由一组Generator函数组成。这些函数负责对HTTP请求进行各种加工，比如生成缓存、指定代理、请求重定向等等。

```
var koa = require('koa');
var app = koa();

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);

```

上面代码中，变量app就是一个Koa应用。它监听3000端口，返回一个内容为Hello World的网页。

app.use方法用于向middleware数组添加Generator函数。

listen方法指定监听端口，并启动当前应用。它实际上等同于下面的代码。
```
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);

```
### 中间件

Koa的中间件很像Express的中间件，也是对HTTP请求进行处理的函数，但是必须是一个Generator函数。而且，Koa的中间件是一个级联式（Cascading）的结构，也就是说，属于是层层调用，第一个中间件调用第二个中间件，第二个调用第三个，以此类推。上游的中间件必须等到下游的中间件返回结果，才会继续执行，这点很像递归。

中间件通过当前应用的use方法注册。

```
app.use(function* (next){
  var start = new Date; // （1）
  yield next;  // （2）
  var ms = new Date - start; // （3）
  console.log('%s %s - %s', this.method, this.url, ms); // （4）
});

```

上面代码中，app.use方法的参数就是中间件，它是一个Generator函数，最大的特征就是function命令与参数之间，必须有一个星号。Generator函数的参数next，表示下一个中间件。

Generator函数内部使用yield命令，将程序的执行权转交给下一个中间件，即yield next，要等到下一个中间件返回结果，才会继续往下执行。上面代码中，Generator函数体内部，第一行赋值语句首先执行，开始计时，第二行yield语句将执行权交给下一个中间件，当前中间件就暂停执行。等到后面的中间件全部执行完成，执行权就回到原来暂停的地方，继续往下执行，这时才会执行第三行，计算这个过程一共花了多少时间，第四行将这个时间打印出来。

下面是一个两个中间件级联的例子。
```
app.use(function *() {
  this.body = "header\n";
  yield saveResults.call(this);
  this.body += "footer\n";
});

function *saveResults() {
  this.body += "Results Saved!\n";
}
```
上面代码中，第一个中间件调用第二个中间件saveResults，它们都向this.body写入内容。最后，this.body的输出如下。

```
header
Results Saved!
footer

```

只要有一个中间件缺少yield next语句，后面的中间件都不会执行，这一点要引起注意。

```
app.use(function *(next){
  console.log('>> one');
  yield next;
  console.log('<< one');
});

app.use(function *(next){
  console.log('>> two');
  this.body = 'two';
  console.log('<< two');
});

app.use(function *(next){
  console.log('>> three');
  yield next;
  console.log('<< three');
});

```

上面代码中，因为第二个中间件少了yield next语句，第三个中间件并不会执行。

如果想跳过一个中间件，可以直接在该中间件的第一行语句写上return yield next。
```
app.use(function* (next) {
  if (skip) return yield next;
})
```
由于Koa要求中间件唯一的参数就是next，导致如果要传入其他参数，必须另外写一个返回Generator函数的函数。
```
function logger(format) {
  return function *(next){
    var str = format
      .replace(':method', this.method)
      .replace(':url', this.url);

    console.log(str);

    yield next;
  }
}

app.use(logger(':method :url'));
```
上面代码中，真正的中间件是logger函数的返回值，而logger函数是可以接受参数的。

### 多个中间件的合并

由于中间件的参数统一为next（意为下一个中间件），因此可以使用.call(this, next)，将多个中间件进行合并。

```
function *random(next) {
  if ('/random' == this.path) {
    this.body = Math.floor(Math.random()*10);
  } else {
    yield next;
  }
};

function *backwards(next) {
  if ('/backwards' == this.path) {
    this.body = 'sdrawkcab';
  } else {
    yield next;
  }
}

function *pi(next) {
  if ('/pi' == this.path) {
    this.body = String(Math.PI);
  } else {
    yield next;
  }
}

function *all(next) {
  yield random.call(this, backwards.call(this, pi.call(this, next)));
}

app.use(all);

```
上面代码中，中间件all内部，就是依次调用random、backwards、pi，后一个中间件就是前一个中间件的参数。

Koa内部使用koa-compose模块，进行同样的操作，下面是它的源码

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

上面代码中，middleware是中间件数组。前一个中间件的参数是后一个中间件，依次类推。如果最后一个中间件没有next参数，则传入一个空函数。
