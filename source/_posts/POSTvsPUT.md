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
我所发现在前端使用上的区别仅仅只有在`<form>`标签的`method`取值中，我们只能定义`POST`和`GET`,不能定义`PUT`，具体可以参考[form - HTML | MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attr-method)。而在xhr中我们可以任意的去`open`我们想要的方法，如果有其他使用上的区别请多指教。

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

`GET`、`HEAD`、`PUT`、`DELETE`、`OPTION`和`TRACE`都享有这个特性，而这里面不包含`POST`。


# 呼之欲出的答案
从以上描述可以看出，`PUT`和`POST`区别关键所在就在于这个幂等的问题上，简而言之，`PUT`是幂等的，而`POST`不是。

`GET`的设计用于获取和检索资源，这种行为应该不产生副作用从而幂等。需要强调的是幂等不代表调取一次或者多次得到的结果不同，而是对资源的状态或者服务器是否产生不一样的影响。比如：`GET http://www.baidu.com/article?id=1`，不会改变id为1的资源状态，不论调用一次还是多次都没有副作用。

`DELETE`方法用于删除资源，有副作用，但满足幂等性。比如：`DELETE http://www.baidu.com/article?id=2`，调用了一次或者多次，都只代表删除id为2的文章，带来的对服务器的影响是相同的。

网上很多答案说：**POST表示创建资源，PUT表示更新资源**,这样说其实并不完全准确，其实二者都可以完成相同的功能，只是在幂等的情况下，不要去使用`POST`而已。创建一篇文章的时候，使用`POST`去完成，比如：`POST http://www.baidu.com/createarticle`每次调用都会新生成一篇文章，id是自增的，这带来的副作用就不一样，所以不幂等。反之，绝大多数情况下的更新资源操作，`PUT http://www.baidu.com/article?id=3`去修改id为3的文章的内容，调用一次和多次都是对3产生影响。

RFC上的说法本质也是这个意思：
> The fundamental difference between the POST and PUT requests is reflected in the different meaning of the Request-URI. The URI in a POST request identifies the resource that will handle the enclosed entity. That resource might be a data-accepting process, a gateway to some other protocol, or a separate entity that accepts annotations. In contrast, the URI in a PUT request identifies the entity enclosed with the request -- the user agent knows what URI is intended and the server MUST NOT attempt to apply the request to some other resource.


# 为了什么？
作为一个前端，其实在使用`POST`和`PUT`的时候我们是可以混用，有时候感觉用错了也不会带来问题，但是我们设计接口时候应该去遵循幂等的原则，从而按照标准的方式去使用它们，如果不遵循规范，可能对于爬虫访问接口就会带来影响。

而在很多基于Rest和Restful架构设计的SOA服务中，需要认真考虑这个问题，由于研究不深就不展开讨论了，这儿有一个知乎上对REST的解释，有兴趣可以戳[怎样用通俗的语言解释REST，以及RESTful？ 知乎](https://www.zhihu.com/question/28557115)