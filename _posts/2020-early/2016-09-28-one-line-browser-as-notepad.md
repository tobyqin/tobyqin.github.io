---
title: 一行代码让浏览器变成记事本
date: 2016-09-28 15:02:46
tags: [javascript, tips]
categories: Tech
---

有时候你为了测试富文本的显示效果，需要新建一个 html 或者 word？

<!-- more -->

太麻烦了。把下面这行代码贴到 Chrome 浏览器地址栏，按下回车，一切都搞定了，so easy！

```
data:text/html,<html contenteditable>
```

现在你可以往浏览器里输入任何内容，爽不？

其实这也不是什么核武器，只是让浏览器直接执行一下它本身就支持的东西。这这个例子我们用的是`Data:`这个 URI 格式，当然你还可以用`javascript:`这个 URI 格式让浏览器立马执行一段 js 脚本，比如这样。

```js
javascript: alert('牛逼大了！')
```
