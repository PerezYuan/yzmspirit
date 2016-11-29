---
title: vue2.0+webpack心路历程
date: 2016-10-25 19:13:30
categories: 前端技术
---

公司后台项目现在由{% link butsalt https://github.com/butsalt true butsalt %}在全权负责，所使用的vue架构也想好好了解下。兴致勃勃开始vue架构的搭建，结果恰好撞到了vue升级2.0的时机，遇到了一下麻烦的问题。把整个心路历程讲一讲

<!-- more -->
# 安装node #

没啥好讲的，是个前端都要明白nodejs和npm的基本用法吧。

# 安装vue #
依靠npm，我们能很轻松的安装各种依赖。
```
$ npm install vue -save
# 安裝babal处理 ES2015
$ npm install babel-core babel-loader babel-plugin-transform-runtime babel-preset-es2015 -save-dev
```
安装完成后，`package.json`文件就变成了这个样子
```
{
  "name": "vue-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "vue": "^2.0.3"
  },
  "devDependencies": {
    "babel-core": "^6.18.0",
    "babel-loader": "^6.2.5",
    "babel-plugin-transform-runtime": "^6.15.0",
    "babel-preset-es2015": "^6.18.0"
  }
}

```

根目录下建立`.babelrc`文件，对babel命令进行配置。
```
{
  "presets": ["es2015"]
}
```

# 构建基本的目录结构 #
```javascript
$ mkdir vue-project
$ cd vue-project
$ npm init -y
```

打开文件夹，看到npm生成的`package.json`文件，这是构建一个项目依赖的基础。我们在根目录新建一个`index.html`

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>my first vue project</title>
</head>
<body>
  <div id="app">{{ message }}</div>
  <script src="dist/bundle.js"></script>
</body>
</html>
```
讲两点
1. 这是一个vue的基本demo，依赖的双向绑定的语法
2. **dist/bundle.js** 是整个项目的入口，依赖`webpack`编译

新建`src`，并新建`main.js`，写入一个最基本的vue demo。
```javascript
import Vue from 'vue'

new Vue({
  el: '#app',
  data: {
      message: "Hello Vue"
  }
})
```
我们完成这一步之后，就只需要靠`webpack`来构建整个项目了。

# 安装webpack #
```
npm install webpack -save-dev
# 一些加载器
npm install css-loader style-loader file-loader url-loader -save-dev
```

编写`webpack.config.js`

```javascript
var path = require('path')
var config = {
  entry: path.join(__dirname, 'src', 'main'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/dist/'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      }
    ]
  }
}

module.exports = config
```

安装全局的`webpack`，开始构建
```
$ npm install webpack -g
$ webpack
```


----------
当我以为一切都好了的时候，打开`index.html`出现了一段warning。
{% img /img/20161025.jpg 1034 37 这是一张图片 %}

原来Vue2.0之后，通过npm package下载的只有runtime-only版本，如果需要使用standaloned的，需要添加webpack配置.
编写`webpack.config.js`

```javascript
var path = require('path')
var config = {
  entry: path.join(__dirname, 'src', 'main'),
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/dist/'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: ['','.js', '.vue'],
    alias: {
      vue: 'vue/dist/vue.js'
    }
  }
}
```
这样就大功告成了！！