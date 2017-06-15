---
title: windows和linux下的路径分隔符问题
date: 2017-06-14 18:12:59
tags:
- linux
categories: 操作系统
---

最近在用node的`path.resolve`和`path.join`处理文件路径的时候，在MAC下能顺利跑起来，到了windows下失效了，查了下原因发现是路径分隔符也就是正斜杠和反斜杠的问题，在这里记录一下。

<!-- more -->

# 不一样的地方

Windows：

{% img /img/20170614_1.png 389 115 windows %}

> 路径分隔采用 “ \ ” 来分隔路径，如上图 **C:\Users\admin**

Linux：

{% img /img/20170614_2.png 460 59 windows %}

> 路径分隔采用 “ / ” 来分隔路径，如上图 **~/workspace/github**

所以当你在不同的操作系统中尝试去获取路径的时候，可能得到的结果不一样，而在js中，反斜杠做转义之后一般得到是“ \\\\ ”

# 问题提出
由于路径我们多数情况下都是采用正斜杠的方式（“/”），所以我们需要把windows下获取的路径做一下处理并且返回如同Linux下的得到的路径，如下路径。

`E:\\workspace\\github\\learn\\javascript\\test.html`

如果采用正则replace

`"E:\\workspace\\github\\learn\\javascript\\test.html".replace('\\/g', '/')`

但是这个方法有太多的弊端和问题，我们也意识到反斜杠是做为转义符存在的，得到的结果并不是我们想要的。

# 解决方案
其实在后端场景中，这已经不是一个新鲜的问题了，node的`path`模块给我们提供了一个`path.sep`用于获取当前环境的路径分隔符。[node的path模块](https://nodejs.org/api/path.html#path_path_sep)也详细阐述着了这个玩意的使用

> Provides the platform-specific path segment separator:
- `\` on Windows
- `/` on POSIX
For example on POSIX:
```javascript
'foo/bar/baz'.split(path.sep);
// Returns: ['foo', 'bar', 'baz']
```
> On Windows:
```javascript
'foo\\bar\\baz'.split(path.sep);
// Returns: ['foo', 'bar', 'baz']
```
> Note: On Windows, both the forward slash (`/`) and backward slash (`\`) are accepted as path segment separators; however, the path methods only add backward slashes (`\`).

所以现在我们在node场景下处理上述字符串就很简单了。

`"E:\\workspace\\github\\learn\\javascript\\test.html".split(path.sep).join('/')`

在很多后端语言中都有对这个正反斜杠的处理，如PHP中的`DIRECTORY_SEPARATOR`和`PATH_SEPARATOR`这两个常量，Python中的`os.pathsep`，这儿就不展开了。
