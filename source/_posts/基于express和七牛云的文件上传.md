---
title: 基于express和七牛云的文件上传
date: 2016-12-20 16:12:25
tags:
- node.js
- express
- 云存储
categories: 前端技术
---

上传图片功能在web程序中很多时候都能遇到，这篇文章将详细阐述基于Express和七牛云如何实现文件的上传。

<!-- more -->
本文将主要分为三个部分对整个功能的实现进行阐述
1. **架构篇**
2. **前后端篇**
3. **七牛云篇**


----------
# 架构篇 #
如果你对Express很熟悉，可以跳过。

Express 是一种保持最低程度规模的灵活 Node.js Web 应用程序框架，为 Web 和移动应用程序提供一组强大的功能。具体API和高级用法请参照[Express官网](http://expressjs.com/ "Express官网")

## （一）构建基础项目 ##
首先我们采用Express为我们提供的生成工具Express application generator进行项目最简单的架构处理。
```javascript
# 全局安装express-generator
$ npm install express-generator -g
# 构建名为app的express项目
$ express app
# 构建项目
$ cd app
$ npm install
```

可以看到如下的目录结构
{% img /img/20161219-1.png 460 355 目录结构 %}

**注意一下两个重要的文件**
1. `bin/www` node项目主要的入口文件，配置node.js的http服务和端口并进行监听。
```javascript
// 设置端口
var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);
// 使用app内的配置启动http服务
var server = http.createServer(app);
// 开始监听
server.listen(port);
server.on('error', onError);
server.on('listening', onListening);
```
2. `app.js` express基本配置文件，完成路由注册，模板指定等基础配置项

于是我们可以简单修改一下监听端口，从3000修改到3001端口
```javascript
var port = normalizePort(process.env.PORT || '3001');
```
由于`package.json`文件中包含了这样一条配置
```
"scripts": {
    "start": "node ./bin/www"
}
```
所以直接在目录命令行下执行，我们就可以在localhost:3001中看到我们的基本页面了
```javascript
$ npm start
 
> app@0.0.0 start C:\Users\admin\Desktop\test博客\app
> node ./bin/www
GET / 200 467.622 ms - 170
GET /stylesheets/style.css 200 8.240 ms - 111
GET /favicon.ico 404 31.661 ms - 1185
```

### 扩展nunjucks模板 ###

由于自身不喜欢`jade`对`html`和扩展方式，所以采用`nunjucks`对模板进行扩展，首先安装`nunjucks`模板

```
$ npm install --save nunjucks
```

在nunjucks的[Getting started](https://mozilla.github.io/nunjucks/getting-started.html "Getting started")章节中有这样一段话

> Using express? Simply pass your express app into configure:
```javascript
var app = express();
 
nunjucks.configure('views', {
    autoescape: true,
    express: app
});
 
app.get('/', function(req, res) {
    res.render('index.html');
});
```

注意两点
1. 获取`nunjucks`对象并配置`app`
2. **渲染模板`jade`不带`.html`后缀，`nunjucks`需要加上**

在app.js中找到对应地方，删除原有的`jade`配置，加上如下代码
```javascript
var nunjucks = require('nunjucks');
 
nunjucks.configure(path.join(__dirname, 'views'), {
  autoescape: true,
  express: app,
  watch: true
});
```

修改原`jade`模板中的内容，并在对应路由处加上`.html`后缀
index.js
```javascript
var express = require('express');
var router = express.Router();
 
/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index.html', { title: 'Express' });
});
 
module.exports = router;
```
app.js
```javascript
// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};
 
  // render the error page
  res.status(err.status || 500);
  res.render('error.html');
});
```
layout.html:
```html
<!doctype html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	<link rel="stylesheet" href="/stylesheets/style.css">    
	<title>{{ title | safe }}</title>
    {% block header %}
    {% endblock %}
</head>
<body>
{% block content %}{% endblock %}
</body>
</html>
```
index.html:
```html
{% extends "layout.html" %}
 
{% block content %}
    <h1>{{ title | safe }}</h1>
    <p>Welcome to {{ title | safe }}</p>
{% endblock %}
```
error.html:
```html
{% extends "layout.html" %}
 
{% block content %}
    error
{% endblock %}
```


## （二）关于路由 ##
我们可以看到，在routes下面有两个文件，分别是`index.js`和`users.js`，他们的结构都类似
```javascript
var express = require('express');
var router = express.Router();
 
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
 
module.exports = router;
```
- `router`对象可以通过`router.METHOD`获取到路由，如`router.get`、`router.post`、`router.put`等。
- 利用中间件中的`res`对象，可以执行渲染模板或者返回具体对象等操作

到此为止我们大致的项目架构就搭建起来了。

----------
# 前后端篇 #

下面我们将来尝试写具体的上传实现代码，大致思路为前端在点击按钮的时候创建一个`input type=file`对象,通过配置的参数完成对象的基本属性配置并点击，并构造一个FormData和xhr对象上传给后端，后端接受到FormData后存在一个缓存的`temp`文件夹中。

## （一）前端实现 ##
废话不多说，直接上代码

index.html
```html
{% extends "layout.html" %}
 
{% block content %}
    <h1>{{ title | safe }}</h1>
    <p>Welcome to {{ title | safe }}</p>
 
    <button id="up">点击上传文件</button>
    <div>
        <p id="note"></p>
        <p id="process"></p>
    </div>
 
    <script>
        var btn = document.getElementById('up'),
            note = document.getElementById("note"),
            prs = document.getElementById("process");
 
        btn.onclick = function() {
            myUpload({
                url: '/upload',
                maxSize: 10,
                multiple: true,
                beforeSend: function(file) {
                    note.innerText = "开始上传...";
                },
                callback: function(res, input) {
                    res = JSON.parse(res);
                    if(res.code == 1) {
                        note.innerText = "上传成功!"
                    } else {
                        console.log(res.msg);
                    }
                    document.body.removeChild(input);
                },
                uploading: function(pre) {
                    prs.innerText = "当前上传进度为：" + pre + "%";
                }
            });
        }
 
        function myUpload(option) {
            var fd = new FormData(),
                xhr = new XMLHttpRequest(),
                input;
            input = document.createElement('input');
            input.setAttribute('id', 'myUploadInput');
            input.setAttribute('type', 'file');
            input.setAttribute('name', 'file');
            if(option.multiple) {
                input.setAttribute('multiple', true);
            }
            document.body.appendChild(input);
            input.style.display = 'none';
            input.click();
            var fileType = ['jpg','png'];
            input.onchange = function() {
            	if(input.files.length == 0) { return false; }
                for(var i = 0; i < input.files.length; i++) {
                	var file = input.files[i];
                	var type = file.name.split('.').pop();
                    if(option.maxSize &&  file.size > option.maxSize * 1024 * 1024){
                        alert('请上传小于' + option.maxSize + 'M的文件');
                        document.body.removeChild(input);
                        return false;
                    }
                    if(fileType.indexOf(type.toLocaleLowerCase()) == -1) {
	                    alert("暂不支持该类型的文件，请重新选择!");
	                    document.body.removeChild(input);
	                    return false;
	                }
	                console.log(fd)
                    fd.append('file' + i, file);
                }
                if(option.beforeSend instanceof Function) {
                    if(option.beforeSend(input) === false) {
                    	document.body.removeChild(input);
                        return false;
                    }
                }
                xhr.open('post', option.url);
                xhr.onreadystatechange = function() {
                    if(xhr.status == 200){
                        if(xhr.readyState == 4) {
                            if(option.callback instanceof Function) {
                                option.callback(xhr.responseText, input);
                            }
                        }
                    } else {
                    	document.body.removeChild(input);
                        console.log("上传失败！");
                    }
                }
                xhr.upload.onprogress = function(event) {
                    var pre = Math.floor(100 * event.loaded / event.total);
                    if(option.uploading instanceof Function) {
                        option.uploading(pre);
                    }
                }
                xhr.send(fd);
            }
        }
    </script>
{% endblock %}
```

大家可以尝试下这段代码，完成一个前端上传的简单实现，这里并不能成功，没有得到服务器的反馈，因为我们还没有配置后端接收请求的路由。这里可以思考两个问题：
1. `input`对象在被创建之后，会反复地创建和销毁，在性能考虑上可以尝试只添加一次DOM元素，做判断后处理
2. 原生的xhr实现可能会有兼容性bug，可以尝试使用`jQuery`的`$.ajax`方法，注意必须设置`processData=false`取消对数据的预处理，`contentType=false`取消默认的Content-Type，以及截取`xhr`对象做`process`处理，具体可以参考[http://www.tuicool.com/articles/RZBJBv](http://www.tuicool.com/articles/RZBJBv "http://www.tuicool.com/articles/RZBJBv")

## （二）后端实现 ##

接下来我们就要开始接收从前端传过来的文章数据了

首先添加路由，由于前端是`post`请求，所以在`index.js`中添加一段路由

```javascript
router.post('/upload', function(req, res, next) {
    res.json({ code : 1 })
});
```

由于默认的express中req.files对象不能直接接收FormData对象，所以这里我们要借助一个中间件的库`connect-multiparty`。[connect-multiparty的github](https://github.com/expressjs/connect-multiparty "connect-multiparty")
```
# 安装connect-multiparty
$ npm install --save connect-multiparty
```

配置上传的目录和中间件
```javascript
const multiparty = require('connect-multiparty');
const path = require('path');
let uploadDir = path.resolve(__dirname, '../temp');
let multipartMiddleware = multiparty({uploadDir});
 
router.post('/upload', multipartMiddleware, function(req, res, next) {
    console.log(req.files);
    res.json({ code : 1 })
});
```

现在我们默认每次上传都是成功的，返回的`code : 1`，下面再试一试上传按钮
```
GET / 304 26.433 ms - -
GET /stylesheets/style.css 304 3.229 ms - -
{ file0:
   { fieldName: 'file0',
     originalFilename: 'f.jpg',
     path: 'C:\\Users\\admin\\Desktop\\test博客\\app\\temp\\FEQHjDOAAhIQtiBKxbROngvq.jpg',
     headers:
      { 'content-disposition': 'form-data; name="file0"; filename="f.jpg"',
        'content-type': 'image/jpeg' },
     size: 80777,
     name: 'f.jpg',
     type: 'image/jpeg' } }
POST /upload 200 25.802 ms - 10

```
就可以在temp目录下看到我们上传的文件了，有没有很激动

{% img /img/20161219-2.png 336 200 上传图片 %}

----------
# 七牛云篇 #
[七牛云](https://www.qiniu.com/ "七牛云")是企业级的云服务提供商，为免费用户提供了10G的存储空间，每个月100万次Get，10万次的Put/Delete完全够用。提供了比较完整的node.js文档。

下面我们来同步七牛云吧。

## （一）注册账号创建对象存储空间 ##
{% img /img/20161219-3.png 800 538 上传图片 %}
{% img /img/20161219-4.png 800 390 上传图片 %}

## （二）配置七牛云 ##

七牛云提供了npm模块安装方案
```
$ npm install --save qiniu
```

我们可以跟着[文档](http://developer.qiniu.com/code/v6/sdk/nodejs.html "文档")，找到配置Key配置 
{% img /img/20161219-5.png 719 385 上传图片 %}

上传代码：

```javascript
router.post('/upload', multipartMiddleware, function(req, res, next) {
    console.log(req.files);
    var qiniu = require("qiniu");
 
    //需要填写你的 Access Key 和 Secret Key
    qiniu.conf.ACCESS_KEY = 'Access_Key';
    qiniu.conf.SECRET_KEY = 'Secret_Key';
 
    //要上传的空间
    bucket = 'tinashy';
  
    //构建上传策略函数
    function uptoken(bucket, key) {
        var putPolicy = new qiniu.rs.PutPolicy(bucket+":"+key);
        return putPolicy.token();
    }
  
    //构造上传函数
    function uploadFile(uptoken, key, localFile) {
        var extra = new qiniu.io.PutExtra();
        qiniu.io.putFile(uptoken, key, localFile, extra, function(err, ret) {
            if(!err) {
                // 上传成功， 处理返回值
                console.log(ret.hash, ret.key, ret.persistentId);
                res.json({ code : 1 });   
            } else {
                // 上传失败， 处理返回代码
                console.log(err);
                res.json({ code : 0, msg : '上传失败' });
            }
        });
    }
 
    for (index in req.files) {
        //上传到七牛后保存的文件名
        key = req.files[index].originalFilename;
        //生成上传 Token
        token = uptoken(bucket, key);
  
        //要上传文件的本地路径
        filePath = req.files[index].path;
 
        //调用uploadFile上传
        uploadFile(token, key, filePath);
    }
});
```

下面就可以看到我刚刚上传的f.jpg文件了，就基本大功告成了！！

{% img /img/20161219-6.png 800 322 上传图片 %}
