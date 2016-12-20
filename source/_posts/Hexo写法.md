---
title: Hexo写法
date: 2016-04-21 19:44:12
tags:
- hexo
categories: 博客
photos:
- http://bruce.u.qiniudn.com/2013/11/27/reading/photos-0.jpg
- http://bruce.u.qiniudn.com/2013/11/27/reading/photos-1.jpg
---

这里会记录一些Hexo和markdown的写法。不定期更新下
<!--more-->
这是**新的开始**，我用hexo创建了第一篇文章。

通过下面的命令，就可以创建新文章
```{bash}
D:\workspace\javascript\nodejs-hexo>hexo new 新的开始
[info] File created at D:\workspace\javascript\nodejs-hexo\source\_posts\新的开始.md
```

## 代码块
{% codeblock .compact http://underscorejs.org/#compact Underscore.js %}
.compact([0, 1, false, 2, ‘’, 3]);
=> [1, 2, 3]
{% endcodeblock %}

## 带有高亮的代码块
```javascript
	var x = 1;
```

```php
	int x = 1;
```
## 链接
{% link 粉丝日志 http://blog.fens.me true 粉丝日志 %}

## 图片
{% img /img/love.jpg 198 220 这是一张图片 %}