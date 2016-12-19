---
title: 基于express和七牛云的上传和读取
date: 2016-12-19 14:12:25
categories: 前端技术
---

上传图片功能在web程序中很多时候都能遇到，这篇文章将详细阐述基于Express和七牛云如何实现文章的上传和读取。

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

