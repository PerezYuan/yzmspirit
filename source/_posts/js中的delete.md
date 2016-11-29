---
title: js中的delete
date: 2016-05-11 16:25:36
tags:
- javascript
- delete
categories: javascript
---

在上一次的code review中，发现了一个delete的问题，在写代码的过程中只是随手写了一个delete方法，却是完全不合理的用法。

首先，`delete`删除成功返回`true`，失败返回`false`。
<!--more-->
#### js代码：

```javascript
function wxCount ($element) {
    this.init($element);
}

wxCount.prototype = {
    init : function(){...}, //初始化方法
    count ： function(){...}, //计算方法
    destroy ： function(){
      delete this;
    } //删除方法
}
```

仔细一看便知，这里的`this`指向的是`function wxCount()`，而在javascript中，`delete`是无法删除一般的变量或者`function`的，永远都会返回`false`。

#### example：

```javascript
<script type="text/javascript">
    x = 1;         // 创建全局属性x
    var y = 2;     // var声明，y为变量
    obj = {
	a : 3,
	b : 4
    };              // 创建全局对象obj，并有a和b两个成员变量

    delete x;       // returns true
               
    delete y;       // returns false 

    delete Math.PI; // returns false 

    delete obj.a; // returns true 
    delete obj.b; // returns true 

    delete obj;   // returns true

    function f() {
        var z = 5;
        var obj2 = {
            c : 6,
            d : 7
        }
        obj3 = {
            e : 8
        }
        delete z;     // returns false
        delete obj2;     // returns false
        delete obj2.c;     // returns true
        delete obj3;     // returns true
    }
</script>	
```
大多数情况下可以理解为：**通过变量或者函数声明的属性不能删除**。

### 下面对几种情况进行简单分析

#### 1.全局属性可以删除
```javascript
    x = 150;
    delete x; //return true

    function f() {
        obj = {
            a : 2
        }
    }

    delete obj; //return true
```
`x`可以理解为全局对象`GLOBAL`的一个属性，即`GLOBAL.x`，可以删除成功。函数中的`obj`也被置为全局属性，同理。

#### 2.通过`var`或者`function`声明的属性不能删除
```javascript
    var x = 150;

    delete x; //return fale

    function y() {}

    delete y; //return false 
```

### 3.内置的属性不能删除
```javascript
    delete Math.PI //reuturn false

    function (param) {
        delete param; //return false
        console.log(param); //1
    }(1);
```

### 4.原型上的属性
```javascript
	function People(){}

	People.prototype.age = 18;

	var Perez = new People();
	Perez.age = 100;

	console.log(Perez.age); //100
	delete Perez.age; //return ture
	console.log(Perez.age); //18
	//返回成功但是并没有什么用，Perez还是会继承原型上的age属性
	
	delete People.prototype.age;
	console.log(Perez.age); //undefined
```

### 5.删除数组中元素
```javascript
    var ipr = ["zhong","du","sun","an","yuan"];

    delete ipr[4];  //return true
    console.log(ipr);  //["zhong","du","sun","an"]
    console.log(ipr.length);  //5
    console.log(4 in ipr);  //false
    console.log(ipr[4]);  //undefined
```

删除数组中元素，数组长度不会发生变化，但是元素实际不存在数组中了。

```javascript
	var ipr = ["zhong","du","sun","an",undefined];
	console.log(4 in ipr);  //ture
```

如果将最后个元素置为undefined，实际还是存在数组中。


关于delete更深的问题和兼容性问题，可以参考
[深入理解JS的delete](http://blog.csdn.net/renfufei/article/details/18965545 "深入理解JS的delete")