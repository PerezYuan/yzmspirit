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
        console.log(this.a);
    }
}
 
var obj = {
    a: "inner",
    say: fn
}
 
obj.say()(); // outer
```

如果认真分析了{% link 函数与this http://www.yzmspirit.com/2017/02/12/this%E4%B8%8E%E5%87%BD%E6%95%B0/ true 函数与this%}中提出的函数引用问题，我们就可以很清晰地明白为什么这儿打印的是`outer`了。
这是因为函数本身被invoke的时候是在全局的执行上下文中，也就是说前一句`obj.say()`只是返回了一个`FunctionDeclaration`，而真正的`obj.say()()`第二个括号才是在对函数进行真正的调用，成为一个`FunctionExpression `所以可以理解为`obj.say().call(window)`。

那么常见有两种方法解决这个`this`指向的问题。

### 方法一
```javascript
var a = "outer";
function fn() {
    var that = this;  // 用一个that来使得this在函数声明时保存下来
    return function() {
        console.log(that.a);
    }
}
 
var obj = {
    a: "inner",
    say: fn
}
 
obj.say()(); // inner
```
这其实打印的就不是在函数invoke时候`this`中的`a`了，而是在函数声明时候的`a`。

### 方法二
```javascript
var a = "outer";
function fn() {
    return function() {
        console.log(this.a);
    }
}
 
var obj = {
    a: "inner",
    say: fn()
}
 
obj.say(); // inner
```
这其实改变了函数invoke的位置，之前obj的say只是引用了`fn`函数，这种改变相当于直接调用了`fn`函数从而使得`this`所在的`context`是obj环境了。


## ES6箭头函数
关于箭头函数的用法，就不赘述了，我们来把最初的例子改成箭头函数的形式：
```javascript
var a = "outer";
function fn() {
    return () => {
        console.log(this.a);
    }
}
 
var obj = {
    a: "inner",
    say: fn
}
 
obj.say()(); // inner
```

哇嚓，怎么打印出来了`inner`？我们所认为的全局invoke的方式怎么错了呢？其实通过简单的查看MDN上关于`arrow function`的介绍，就可以知道一个关键点`No binding of this`。

> Until arrow functions, every new function defined its own this value (a new object in the case of a constructor, undefined in strict mode function calls, the context object if the function is called as an "object method", etc.). This proved to be annoying with an object-oriented style of programming.

大致的意思就是，到箭头函数为止，函数才拥有了自己本来的`this`值，这个是一种面向对象编程的概念，`this`就不会根据函数被invoke的场景改变了。

那么要解释上述箭头函数例子，请看下面一个概念。

### 词法作用域(lexical scope)
更加准确地来说，箭头函数中的`this`是指向当前的词法作用域的，什么是词法作用域？简单地说，词法作用域就是定义在词法阶段的作用域，是由写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变。也就是说无论函数在哪儿被调用，也无论怎么被调用，他的词法作用域是有他被声明的时候所决定的。
来看下面一个例子：

```javascript
var a = "outer";
var b = () => {
    console.log(this.a);
}
 
var obj = {
    a: "inner"
}
 
b.call(obj);  // outer;
```
很明显，即使我们用`call()`方法显式制定当前调用函数的对象是`obj`也无法改变函数`b`中`this`的指向，这是用`b`在声明的时候就决定了，当函数`b`声明的时候，他是处于全局作用域的。

参考文献：{% link ES6 arrow functions, syntax and lexical scoping https://toddmotto.com/es6-arrow-functions-syntaxes-and-lexical-scoping/ true ES6 arrow functions, syntax and lexical scoping%}


## 更加正确的理解
其实了解了上述关于箭头函数`this`的解释，基本就了解了箭头函数关于`this`的坑了。但是知识是无穷尽的，如果你把箭头函数的`this`理解为箭头函数内部中创建的一个变量的话，那就大错特错了。他的工作方式并不是`var that = this`这种理解（that为箭头函数的this）,其实MDN上的说法是最准确的`No binding of this`和`No binding of arguments`

其实，箭头函数在自身其实并没有创建一个`this`，而是沿着像作用域链一样去寻找当前上下文的`this`，这也是词法作用域这种解释的根本所在，如下代码，理解有几次`this`的绑定操作被执行：
```javascript
var a = "outer";
var foo = () => {
    return () => {
        return () => {
            return () => {
                console.log(this.a);
            };
        };
    };
}

foo.call( { a: "inner" } )()()(); // outer
```

如果你认为有4次（一次全局，3次return），那么结果是否定的，其实只有一次全局的this绑定，当到`console`代码段的时候，其实根本就没有执行新的`this`绑定，而是按照词法作用域向上查找，找到了全局中`this = window`的绑定，所以`this.a`才会打印`outer`。

那么再来解释最初的例子。
```javascript
var a = "outer";
function fn() {
    return () => {
        console.log(this.a);
    }
}
 
var obj = {
    a: "inner",
    say: fn
}
 
obj.say()(); // inner
```
其实箭头函数中的`this`也是向上查找，找到了`fn()`函数的`this`之后就直接返回了，而此时的`fn()`函数是被`obj`所invoke得，固然`this.a`就打印了`inner`。