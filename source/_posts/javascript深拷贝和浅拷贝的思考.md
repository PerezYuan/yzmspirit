---
title: javascript深拷贝和浅拷贝的思考
date: 2016-05-16 14:51:20
tags:
- js
- 拷贝
categories: 前端技术
---

在做一个数据对象的时候需要拷贝一下对象，突然想到以前在js中存在的深拷贝和浅拷贝，这里复习一下也想深究一下。
**首先看一段代码**

```javascript
	var obj = { a : 1, arr: [1,2]}
	var obj1 = obj;            //浅拷贝
	var obj2 = deepCopy(obj);  //深拷贝	
```
<!--more-->
js是把**对象**和**数组**存储在某一块内存区域的，所以浅拷贝会将`obj`和`obj1 `指向同一块内存地址。而深拷贝一般都是开辟一块新的内存地址，将原对象的各个属性逐个复制出去。简单图示如下（图转自知乎）：

{% img /img/20160519.jpg 600 225 这是一张图片 %}

那么就很容易理解如下代码了
```javascript
obj.a = 10;
console.log(obj1.a); //输出10
console.log(obj2.a); //输出1
```

## 介绍一些简单的解决深拷贝的方法。

#### 1.单纯的数组可以使用slice()方法进行拷贝

```javascript
var arr = [1,2,3,4,5];
var arr2 = arr;
var arr3 = arr.slice(0);
arr[4] = 5656565;
console.log(arr);     //[1,2,3,4,5656565]
console.log(arr2);    //[1,2,3,4,5656565]
console.log(arr3);    //[1,2,3,4,5]
```

#### 2.借助JSON全局对象

针对纯的JSON对象拷贝可以使用`JSON.stringify`和`JSON.parse`方法，原理也很简单，将对象转换为JSON字符串在重新序列化一下。这是一个简单讨巧的方法。但是这个方法只能针对诸如`Number`, `String`, `Boolean`, `Array`, 扁平对象等，对如`Date`，`RegExp`这类对象并不适用。

```javascript
function jsonClone(obj) {
    return JSON.parse(JSON.stringify(obj));
}
```

#### 3.利用jQuery的$.extend()方法

jQuery的`$.extend()`可以将多个对象合并并返回合并后的对象，我们可以将需要拷贝的数据和`{}`进行合并。值得注意的是当出现嵌套对象的情况下，`$.extend()`提供了第一个参数来进行递归的深拷贝。

```javascript
		var people = {
		    name : 'perez',
		    age : 26,
		    friends : {
		        people : {
		            name : 'butsalt'
		        }
		    }
		}
 
		var people2 = $.extend({},people);
		var people3 = $.extend(true,{},people);
		 
		people.friends.people.name = 'butsaltme';
		console.log(people2.friends.people.name); //butsaltme
		 
		console.log(people.friends.people === people2.friends.people);  //true
		console.log(people.friends.people === people3.friends.people);  //false
```

***

网上有段如下代码，能解决大多数情况。
```javascript
var cloneObj = function(obj){
    var str, newobj = obj.constructor === Array ? [] : {};
    if(typeof obj !== 'object'){
        return;
    } else if(window.JSON){
        str = JSON.stringify(obj), //系列化对象
        newobj = JSON.parse(str); //还原
    } else {
        for(var i in obj){
            newobj[i] = typeof obj[i] === 'object' ? 
            cloneObj(obj[i]) : obj[i]; 
        }
    }
    return newobj;
};
```

参考资料：{% link 深入剖析 JavaScript 的深复制 http://jerryzou.com/posts/dive-into-deep-clone-in-javascript/ 深入剖析 JavaScript 的深复制 %}