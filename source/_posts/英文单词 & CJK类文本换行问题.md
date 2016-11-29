---
title: 英文单词 & CJK类文本换行问题
date: 2016-07-04 16:36:23
tags:
- CSS
- 文本换行
categories: 前端技术
---

在前端开发中，提到英文单词换行首先肯定是想到文本超过了其容器导致的自动换行，以及相关处理的word-break和word-wrap属性，现在对CSS中的换行方法进行一个简单的整理和总结。

<!-- more -->

# 一.word-break #

单独使用word-break能解决近乎所有的单词换行问题。`word-break`有三个属性

## 1. word-break:normal ##
使用浏览器默认的换行处理方法，这回导致很多问题。在英文段落中入如果遇到较长的单词，它的长度超出容器宽度，浏览器就会自动将这个单词移到下一行，途中的China就被换行了。其次，当某个单词本身的长度超过了容器的宽度，容器对单词的约束力就没有了，最终导致了图中的dddd...被当做了一个单词，溢出了容器。

![Font Boosting](/img/20160704165831.jpg)

## 2. word-break:break-all ##
于是我们机智的采用了`word-break:break-all`来处理这种情况，以保证在某些不合法英文长串所引起的页面变形。但是这样会导致原本合理的英文串变成了我们不想要看到的换行样子，如图中的China的n和a被分开了。

![Font Boosting](/img/20160704173446.jpg)

## 3. word-break:keep-all ##
绝大多数情况下，我们不会用到keep-all，这个只对我们的CJK类语言起作用，CJK类语言是中日韩统一表意文字[（维基百科）](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97 "中日韩统一表意文字")，它会让CJK类语言不换行，但不会影响到其他的文本。

![Font Boosting](/img/20160704174433.jpg)

[（运行Demo）](http://www.yzmspirit.com/code/word-break.html)


# 二.word-wrap #

要解决上述第二点中将本来单词给打断的问题，CSS3引入了一个新的属性`word-wrap`，这个值的取值也很简单，只有normal | break-word，它会将长单词或 URL 地址换行到下一行。一般情况下，我们只需要在单词本身长度超过容器宽度的时候，对其进行换行处理，所以在网页设计中，只使用一个`word-wrap:brak-word`就能够满足网页上最基本的排版了。

![Font Boosting](/img/20160705160822.jpg)

[（运行Demo）](http://www.yzmspirit.com/code/word-wrap.html)

# 三.hyphens #

人总是不满足的，为什么我们的网页不能像一些书本一样，再遇到单词换行的时候，给没写完的单词加上一个'-'呢？其实是由解决方案的，那就是CSS3的属性`hyphens`。`hyphens`**在目前的情况下被支持还是很有限的，我们伟大的Chrome就不支持这个属性，所以我们在使用它的时候要格外小心**。

```css
-webkit-hyphens: auto;
-moz-hyphens: auto;
-ms-hyphens: auto;
hyphens: auto;
```

文档的编码也必须是能支持`hyphens`的语种

```HTML
<html lang="en">
```

![Font Boosting](/img/20160705162435.jpg)

[（运行Demo）](http://www.yzmspirit.com/code/hyphens.html)