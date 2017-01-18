---
title: 在ajax中动态创建input type='file'问题
date: 2017-1-17 15:41:28
tags:
- javascript
categories: 前端技术
---

自己在写站点的时候出现了一个交互：给用户提供上传功能，但是在用户实现上传之前会对用户做一个认证，判断用户是否有资格上传，大致流程图如下

<!-- more -->
{% img /img/20170117.png 435 589 流程图 %}

**实现这个需求，首先创建一个基本的上传功能**
1. html代码
```html
<button id="click">
  click me
</button>
```
2. js代码
```javascript
$('#click').on('click',function(){
  $('<input type="file">').click();
})
```

----------

> 此处**不考虑**`<input type="file">`change之后的功能

显然，当我们点击按钮的时候，就会弹出一个文件选择框，于是我开始加上我的`ajax`验证
```javascript
function getAnswer() {
  /* 省略 */
  return answer;
}
 
$('#click').on('click',function(){
  var answer = getAnswer();
  $.ajax({
    url: '/getAnswer',
    data: answer,
    success: function(res){
        if(res.status == 1) {
          /* 成功 */
          $('<input type="file">').click();
        } else {
          /* 失败 */
        }
    },
    dataType: 'json'
  })
})
```

此时我想要达到的效果是在我点击按钮时,调用`getAnswer`方法，再去请求后台接口，当验证成功时候，弹出选择文件框。但是一试发现并不能弹出框，如果当DOM ready之后直接`$('<input type="file">').click();`也是不能成功弹出选择文件框的，这是怎么一回事呢？？？
demo：[demo](https://jsfiddle.net/perezyuan/16jtcpo2/6/ "demo")

万能的stackoverflow给了我答案：[Trigger click on input=file on asynchronous ajax done()](http://stackoverflow.com/questions/29728705/trigger-click-on-input-file-on-asynchronous-ajax-done "Trigger click on input=file on asynchronous ajax done()")


----------
结论：
1. 简单的来看，只用加上一行`async: false`即可解决问题。
2. 为什么改变同步异步方式能打开选择文件框呢，这是w3c的一个安全原因，异步的请求是`untrusted events`，在`<input type="file">`这个元素中会阻止浏览器进行下一步的操作，而改为同步方式，可以理解为是用户的`click`事件触发了`input`元素的后一步操作，stackoverflow的回答中已经讲得非常清楚了。
3. 这种交互不是推荐的交互，最好是在最先就做好权限验证而不是到最后一步上传文件了再去做验证。
> [trusted-events](https://www.w3.org/TR/2012/WD-DOM-Level-3-Events-20120614/#trusted-events "trusted-events")：Most untrusted events should not trigger default actions, with the exception of click or DOMActivate events