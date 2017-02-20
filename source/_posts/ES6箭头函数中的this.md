---
title: 进击的ES6箭头函数和this
date: 2017-2-23 22:37:16
tags:
- js
- this
- es6
categories: 前端技术
---

关于函数与`this`之间的关系，已经在上一篇博客中简单讲解了，欢迎戳{% link 函数与this http://www.yzmspirit.com/2017/02/12/this%E4%B8%8E%E5%87%BD%E6%95%B0/ true 函数与this%}，接下来我们要来细细看一下ES6中的箭头函数还有它所带来的`this`问题。

<!-- more -->

## 一个闭包引起的思考
```javascript
var a = "outer";
function fn() {
    return function() {
        console.log(this.a)
    }
}
 
var obj = {
    a: "inner",
    say: fn
}
 
obj.say()(); // outer
```