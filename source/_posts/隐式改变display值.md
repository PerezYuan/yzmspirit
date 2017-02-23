---
title: 隐式改变display值
date: 2017-2-17 21:15:48
tags:
- css
categories: 前端技术
---

今天写布局玩的时候，突然想到一个知识点，当一个`inline`元素被`position: absolute`的时候，会隐式地改变其`display`属性，当时就和同事讨论了一下，因为我记得我以前看到过博客中有提到，隐式改变后`display`是`inline-block`，而同事说的则是`block`，于是一个吃屎的赌博就产生了。。。。。那么你一定知道是谁吃屎了吧~

<!-- more -->

立马demo走起，创建一个`inline`元素:
```javascript
<span>我是一个inline元素</span>
```

然后打开控制台，改变他的`display`属性。结果固然明显得展示在我的眼前：

{% img /img/20170217.gif 800 600 %}

我的心当时碎了，在我的脑海中我总觉得在哪儿见过最后是`inline-block`，其实仔细想想，如果是`inline-block`，那么类`inline`元素的`baseline`规则就不好理解了？于是搜索起来，结果搜索出来的结果大部分是下面这句话：

{% img /img/20170217_1.png 943 155 baidu图1 %}
张鑫旭大神也发表过一篇文章，{% link css-相对绝对定位系列（一） http://www.zhangxinxu.com/wordpress/2010/12/css-%E7%9B%B8%E5%AF%B9%E7%BB%9D%E5%AF%B9%E5%AE%9A%E4%BD%8D%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%80%EF%BC%89/ true css-相对绝对定位系列（一）%}，当中提到了一个概念
> 包裹性
> 包裹性换种说法就是让元素inline-block化，例如一个div标签默认宽度是100%显示的，但是一旦被absolute属性缠上，则100%默认宽度就会变成自适应内部元素的宽度。

其实张大神的措辞很巧妙，张含韵妹子的demo也非常直观，不能单纯用`block`和`inline-block`来定义，而是说了句**`inline-block`化**，虽然是`display: block`，但展现行为上来更像`inline-block`。

#### 更细的思考
其实，这些在w3c标准中去追寻，就能得到一些更加明确的答案：
{% link Relationships between 'display', 'position', and 'float' https://www.w3.org/TR/CSS21/visuren.html#dis-pos-flo true Relationships between 'display', 'position', and 'float'%}

#### 总结
我们再看待一个问题的时候，不能单单的去看一个博客的阐述或者一个所谓搜索出来的结果，更多的去思考，从标准入手去理解，才能做到不被一些东西给蒙蔽，好了，我去吃屎了。。。。