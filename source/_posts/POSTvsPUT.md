---
title: POST vs PUT
date: 2017-06-06 06:52:11
tags:
- http
categories: 前端技术
---

最近接触到了一个问题，在`http`原语中，`POST`与`PUT`有什么区别？以前绝大部分情况下都只关心了`GET`和`POST`之间的区别，而没有去深入了解过其他方法的区别，深感自己学习知识的不扎实，于是想好好总结下。

<!-- more -->

# 前端使用上的区别
我所发现在前端使用上的区别仅仅只有在`<form>`标签的`method`取值中，我们只能定义`post`和`get`,不能定义`put`，具体可以参考[form - HTML | MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attr-method)。而在xhr中我们可以任意的去`open`我们想要的方法，如果有其他使用上的区别请多指教。

既然使用上没什么大的区别，同时这两个方法都可以去处理服务器上的资源，那么为什么要定义两个原语呢？

# RFC
第一个想到的当然是直接去[RFC](https://tools.ietf.org/html/rfc2616#section-9)的文档中寻求答案，在文档的第九章节，详细阐述了HTTP/1.1中各个方法的定义，我们从中提取几个比较重要的关键词。

## 1. Safe Method（安全的方法）
核心是以下这段话：
> In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe". This allows user agents to represent other methods, such as POST, PUT and DELETE, in a special way, so that the user is made aware of the fact that a possibly unsafe action is being requested.
   
两个核心点：
- `GET`和`HEAD`方法不应该具有除了检索以外其他的意义，标准约定的是在去查看或者搜索服务器资源时候，使用这两个方法，这样来看，这两个方法应当是被认为安全的。
- 检索资源这种行为也可以发生在当用户使用`POST`、`PUT`、`DELETE`这类方法的请求的时候，这个时候用户可能就请求了一个不安全的动作

## 2.Idempotent Method(幂等的方法)
规范中是这样描述的：
> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.The methods GET, HEAD, PUT and DELETE share this property. Also, the methods OPTIONS and TRACE SHOULD NOT have side effects, and so are inherently idempotent.

从定义上来看，幂等性是各个方法的一种属性，表明使用这种方法一次或者多次带来的`side-effects`（这个直译过来为副作用，但并不是传统意义上的负面作用，具体可以参考[怎么翻译 side effects 好？知乎](https://www.zhihu.com/question/30779564)）是相同的。

