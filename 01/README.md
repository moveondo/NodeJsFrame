##概述


Express是目前最流行的基于Node.js的Web开发框架，可以快速地搭建一个完整功能的网站。

Express上手非常简单，首先新建一个项目目录，假定叫做hello-world。

>$ mkdir hello-world

进入该目录，新建一个package.json文件，内容如下。
```
{
  "name": "hello-world",
  "description": "hello world test app",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "express": "4.x"
  }
}
```


上面代码定义了项目的名称、描述、版本等，并且指定需要4.0版本以上的Express。

然后，就可以安装了。

>$ npm install

执行上面的命令以后，在项目根目录下，新建一个启动文件，假定叫做index.js。

```
var express = require('express');
var app = express();

app.use(express.static(__dirname + '/public'));

app.listen(8080);
```
然后，运行上面的启动脚本。

>$ node index

现在就可以访问http://localhost:8080，它会在浏览器中打开当前目录的public子目录（严格来说，是打开public目录的index.html文件）。如果public目录之中有一个图片文件my_image.png，那么可以用http://localhost:8080/my_image.png访问该文件。

你也可以在index.js之中，生成动态网页。
```
// index.js

var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello world!');
});
app.listen(3000);

```

然后，在命令行下运行启动脚本，就可以在浏览器中访问项目网站了。

>$ node index

上面代码会在本机的3000端口启动一个网站，网页显示Hello World。

启动脚本index.js的app.get方法，用于指定不同的访问路径所对应的回调函数，这叫做“路由”（routing）。上面代码只指定了根目录的回调函数，因此只有一个路由记录。实际应用中，可能有多个路由记录。

```
// index.js

var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello world!');
});
app.get('/customer', function(req, res){
  res.send('customer page');
});
app.get('/admin', function(req, res){
  res.send('admin page');
});

app.listen(3000);
```

这时，最好就把路由放到一个单独的文件中，比如新建一个routes子目录。

```
// routes/index.js

module.exports = function (app) {
  app.get('/', function (req, res) {
    res.send('Hello world');
  });
  app.get('/customer', function(req, res){
    res.send('customer page');
  });
  app.get('/admin', function(req, res){
    res.send('admin page');
  });
};
```

然后，原来的index.js就变成下面这样。

```
// index.js
var express = require('express');
var app = express();
var routes = require('./routes')(app);
app.listen(3000);
```
