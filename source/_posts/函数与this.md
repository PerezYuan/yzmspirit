---
title: 函数与this
date: 2017-2-12 15:10:12
tags:
- js
- this
categories: 前端技术
---

今天看到同事做的一个关于`this`的总结，深思想了一下这方面的知识，也感觉到某些知识掌握得还不算牢固，在这儿做一个比较全面的总结吧。

<!-- more -->

## 1.定义一个函数

首先，我们了解的最常见的函数定义方法有2种，如下所示：

```javascript
// 方法1 函数声明式(FunctionDeclaration)
function fn() {}
// 方法2 函数表达式(FunctionExpression)
var fn = function() {}
```

这里衍生了第一个知识点——**声明提前**，什么是声明提前呢？其实就是js的解析器会将声明一个变量的代码解析到该变量执行代码之前，如下代码打印的是`number`而不是`undefined`。
```javascript
var a = 1;
var a;
 
console.log(a); // number
```

所以有了这个概念 我们就更能理解下面代码为什么`test1`不报错而`test2`报错了：
```javascript
test1(); // 相当于在这行代码之前已经存在一个 var test1了
 
function test1() {
    alert(1);
}
 
test2(); // TypeError: test2 is not a function
 
var test2 = function() {
    alert(1);
}
```

## 2.匿名函数

了解完了函数的定义，我们再来看一下另外一个函数有趣的地方：**匿名函数**

```javascript
(function() {
    alert(1);
})()
```
匿名函数(anonymous)顾名思义，就是没有名字的函数。我们上述的函数定义中，方法二就是用一个变量来接收匿名函数的，那么问题来了，为什么我们去掉双括号之后，匿名函数就不会自执行反而报错了呢？
```javascript
// Error
function() {
    alert(1);
}()
```

我们来看下面一句话（摘自stackoverflow）：
> It doesn't work because it is being parsed as a `FunctionDeclaration`, and the name identifier of function declarations is mandatory.
> When you surround it with parentheses it is evaluated as a `FunctionExpression`, and function expressions can be named or not.

回到函数的定义上来，本质上来讲，匿名函数`function() {}`是叫做函数定义式(FunctionDeclaration)，执行`typeof function(){}`也会返回"function"，但是`FunctionDeclaration`是不能直接执行的，需要将其转换成为`FunctionExpression`才能执行，而我们上述函数定义中讲到的`var test2 = function() {}`就是一种将其转换为`FunctionExpression`的方式。

> The Parentheses (formally called the Grouping Operator) can surround only expressions, and a function expression is evaluated.

所以，括号就能完成匿名函数转换的任务从而使得其可以执行。

----------
See more:
> the Comma operator can also handle only expressions.
```javascript
0,function(){ return 1 }()
=> 1
```

## 3.this
扯了半天，终于到`this`了，这个概念也是js中坑比较多的，但是但凡知道了其中的原理，关于`this`的问题就迎刃而解了。

### 1.执行上下文(Execution context)
每一个函数都有自己的执行上下文，并且每个执行上下文中都有它自己的变量对象，用于存储执行上下文中的变量 、函数声明 、函数参数，这解释了js如何找到我们定义的函数和变量。

**那么`this`指向哪儿，说白了就是产生当前的执行上下文的那个对象，在函数中来说就是调用函数的那个对象**

直接在浏览器的`console`中打印`this`，返回的就是`window`对象。

### 2.块级作用域
函数能创建出新的作用域(scope)，这是if(),for()等办不到的，所以js是没有块级作用域的。
```C
// C语言  
#include <stdio.h>  
void main() {
    if(true) {  
        int i = 3;  
    }  
    printf("%d/n", i);  
}  
```
在C语言中，上述代码是会报错的，因为`i`是在`if`的块级作用域中创建的，所以`printf`并不能再上下文中找到变量`i`，但如下的js代码就能准确地执行：
```javascript
// js语言
if(true) {  
    var i = 3;  
}  
console.log(i); // 3 
```
### 3.函数调用(Invoking a JavaScript Function)
我们现在了解了上述两个概念之后，现在我们要认真地判断当前调用函数的对象了，这里就不得不提几种函数调用的方法从而跟深刻得了解`this`的指向。

1.函数直接调用(Invoking a Function as a Function)
2.作为对象方法调用(Invoking a Function as a Method)
3.作为构造函数调用(Invoking a Function with a Function Constructor)
4.`call()` && `apply()`调用(Invoking a Function with a Function Method)

前3种其实都是第4种的语法糖。

#### 1.函数直接调用
```javascript
var a = 1;
function fn() {
    console.log(this); // Window
    console.log(this.a);
}
 
fn();
```
#### 2.作为对象方法调用
```javascript
var a = 1000;
var obj = {
    a: 1,
    say: function() {
        console.log(this); // obj
        console.log(this.a);
    }
}
 
obj.say();  // 1
```

#### 3.作为构造函数调用
```javascript
var x = 1000;
function fn(arg) {
    console.log(this); // fn
    this.x = arg;
}
 
var obj = new fn(2);
console.log(obj.x); // 2
```

#### 4.`call()` && `apply()`调用
```javascript
function fn() {
    console.log(this); // b
    console.log(this.a);
}
 
var b = {
    a: 5
}
 
fn.call(b);  // 5
```

解释一下：
- 直接函数调用可以看做是`fn.call(window, undefined)`，自然`this`便指向了`window`。
- 方法调用可以看做是`say.call(obj)`，同理，`this`便指向了`obj`。
- 构造函数，其实是使用`new`方法构造出了一个新的对象`obj`，也可以看做是`fn.call(obj)`，`this`便指向了`obj`。

以上都是比较基础的问题了，稍微巩固了下，关于ES6的箭头函数涉及到的`this`问题还没想得特别全面和透彻，再仔细看下会在下面博客中讲咯~