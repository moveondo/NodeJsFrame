### Express.Router用法

从Express 4.0开始，路由器功能成了一个单独的组件Express.Router。它好像小型的express应用程序一样，有自己的use、get、param和route方法。

#### 基本用法

首先，Express.Router是一个构造函数，调用后返回一个路由器实例。然后，使用该实例的HTTP动词方法，为不同的访问路径，指定回调函数；最后，挂载到某个路径。
```
var router = express.Router();

router.get('/', function(req, res) {
  res.send('首页');
});

router.get('/about', function(req, res) {
  res.send('关于');
});

app.use('/', router);
```

上面代码先定义了两个访问路径，然后将它们挂载到根目录。如果最后一行改为app.use(‘/app’, router)，则相当于为/app和/app/about这两个路径，指定了回调函数。

这种路由器可以自由挂载的做法，为程序带来了更大的灵活性，既可以定义多个路由器实例，也可以为将同一个路由器实例挂载到多个路径。

#### router.route方法

router实例对象的route方法，可以接受访问路径作为参数。

```
var router = express.Router();

router.route('/api')
	.post(function(req, res) {
		// ...
	})
	.get(function(req, res) {
		Bear.find(function(err, bears) {
			if (err) res.send(err);
			res.json(bears);
		});
	});

app.use('/', router);
```
#### router中间件

use方法为router对象指定中间件，即在数据正式发给用户之前，对数据进行处理。下面就是一个中间件的例子。
```
router.use(function(req, res, next) {
	console.log(req.method, req.url);
	next();	
});
```

上面代码中，回调函数的next参数，表示接受其他中间件的调用。函数体中的next()，表示将数据传递给下一个中间件。

注意，中间件的放置顺序很重要，等同于执行顺序。而且，中间件必须放在HTTP动词方法之前，否则不会执行。

#### 对路径参数的处理

router对象的param方法用于路径参数的处理，可以
```
router.param('name', function(req, res, next, name) {
	// 对name进行验证或其他处理……
	console.log(name);
	req.name = name;
	next();	
});

router.get('/hello/:name', function(req, res) {
	res.send('hello ' + req.name + '!');
});
```

上面代码中，get方法为访问路径指定了name参数，param方法则是对name参数进行处理。注意，param方法必须放在HTTP动词方法之前。

### app.route

假定app是Express的实例对象，Express 4.0为该对象提供了一个route属性。app.route实际上是express.Router()的缩写形式，除了直接挂载到根路径。因此，对同一个路径指定get和post方法的回调函数，可以写成链式形式。
```
app.route('/login')
	.get(function(req, res) {
		res.send('this is the login form');
	})
	.post(function(req, res) {
		console.log('processing');
		res.send('processing the login form!');
	});
 ```
 
上面代码的这种写法，显然非常简洁清晰。

### 上传文件

首先，在网页插入上传文件的表单。
```
<form action="/pictures/upload" method="POST" enctype="multipart/form-data">
  Select an image to upload:
  <input type="file" name="image">
  <input type="submit" value="Upload Image">
</form>
```
然后，服务器脚本建立指向/upload目录的路由。这时可以安装multer模块，它提供了上传文件的许多功能。
```
var express = require('express');
var router = express.Router();
var multer = require('multer');

var uploading = multer({
  dest: __dirname + '../public/uploads/',
  // 设定限制，每次最多上传1个文件，文件大小不超过1MB
  limits: {fileSize: 1000000, files:1},
})

router.post('/upload', uploading, function(req, res) {

})

module.exports = router
```

上面代码是上传文件到本地目录。下面是上传到Amazon S3的例子。

首先，在S3上面新增CORS配置文件。
```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```
上面的配置允许任意电脑向你的bucket发送HTTP请求。

然后，安装aws-sdk。

>$ npm install aws-sdk --save

下面是服务器脚本。
```
var express = require('express');
var router = express.Router();
var aws = require('aws-sdk');

router.get('/', function(req, res) {
  res.render('index')
})

var AWS_ACCESS_KEY = 'your_AWS_access_key'
var AWS_SECRET_KEY = 'your_AWS_secret_key'
var S3_BUCKET = 'images_upload'

router.get('/sign', function(req, res) {
  aws.config.update({accessKeyId: AWS_ACCESS_KEY, secretAccessKey: AWS_SECRET_KEY});

  var s3 = new aws.S3()
  var options = {
    Bucket: S3_BUCKET,
    Key: req.query.file_name,
    Expires: 60,
    ContentType: req.query.file_type,
    ACL: 'public-read'
  }

  s3.getSignedUrl('putObject', options, function(err, data){
    if(err) return res.send('Error with S3')

    res.json({
      signed_request: data,
      url: 'https://s3.amazonaws.com/' + S3_BUCKET + '/' + req.query.file_name
    })
  })
})

module.exports = router
```

上面代码中，用户访问/sign路径，正确登录后，会收到一个JSON对象，里面是S3返回的数据和一个暂时用来接收上传文件的URL，有效期只有60秒。

浏览器代码如下。
```
// HTML代码为
// <br>Please select an image
// <input type="file" id="image">
// <br>
// <img id="preview">

document.getElementById("image").onchange = function() {
  var file = document.getElementById("image").files[0]
  if (!file) return

  sign_request(file, function(response) {
    upload(file, response.signed_request, response.url, function() {
      document.getElementById("preview").src = response.url
    })
  })
}

function sign_request(file, done) {
  var xhr = new XMLHttpRequest()
  xhr.open("GET", "/sign?file_name=" + file.name + "&file_type=" + file.type)

  xhr.onreadystatechange = function() {
    if(xhr.readyState === 4 && xhr.status === 200) {
      var response = JSON.parse(xhr.responseText)
      done(response)
    }
  }
  xhr.send()
}

function upload(file, signed_request, url, done) {
  var xhr = new XMLHttpRequest()
  xhr.open("PUT", signed_request)
  xhr.setRequestHeader('x-amz-acl', 'public-read')
  xhr.onload = function() {
    if (xhr.status === 200) {
      done()
    }
  }

  xhr.send(file)
}
```
上面代码首先监听file控件的change事件，一旦有变化，就先向服务器要求一个临时的上传URL，然后向该URL上传文件。
