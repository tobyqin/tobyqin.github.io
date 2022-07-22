---
title: 如何使用Python优雅地处理时间数据
categories: [Tech]
tags: [python,arrow,datetime,pypi]
date: 2017-09-26
---

Python处理时间数据的好方法。

<!-- more -->

## 缘起

很多时候我们不得不和时间打交道，但在Python标准库中处理时间的模块其实设计的不是很友好，为什么我会这么说？因为我相信大部分人几乎每次在处理时间数据时一而再，再而三的去查文档，比如时间和文本格式互转，时间增减等看起来非常基本的操作，在Python中处理起来并不轻松。

最要命的是，在Python标准库中居然有两个名字差不多的模块可以处理时间，一个叫time，另外一个叫datetime，里面提供了类似的方法但是两个完全不是一回事。到这还没完，标准库里还有一个叫calendar的模块，也是用来处理时间的。你是不是纠结到底该用哪一个？

今天我不带大家去理解他们三者的关系，因为现在你记住了不代表你以后不会忘记。今天的主角是一个优雅到我不能放弃的时间处理库 - arrow。

## 简介

arrow是一个专门处理时间和日期的轻量级Python库，它提供了一种**合理、智能**的方式来创建、操作、格式化、转换时间和日期。

## 安装

```shell
pip install arrow
```

## 使用

我们直接看代码，注释既分割线。

```python
>>> import arrow

# 获取当前时间
>>> utc = arrow.utcnow()
>>> utc
<Arrow [2017-05-11T21:23:58.970460+00:00]>

# 调整时间
>>> utc = utc.shift(days=+1, hours=-1)
>>> utc
<Arrow [2017-05-12T20:23:58.970460+00:00]>

# 修改时间
>>> utc.replace(hour=4, minute=40)
<Arrow [2017-05-12T04:40:58.970460+00:00]>

# 转换时区
>>> local = utc.to('US/Pacific')
>>> local
<Arrow [2017-05-11T13:23:58.970460-07:00]>

# 从文本转为时间对象
>>> arrow.get('2017-05-11T21:23:58.970460+00:00')
<Arrow [2017-05-11T21:23:58.970460+00:00]>

>>> arrow.get(1367900664)
<Arrow [2017-05-07T04:24:24+00:00]>

>>> arrow.get('June was born in May 1980', 'MMMM YYYY')
<Arrow [1980-05-01T00:00:00+00:00]>

# 获取时间戳
>>> local.timestamp
1368303838

# 格式化输出
>>> local.format()
'2017-05-11 13:23:58 -07:00'

>>> local.format('YYYY-MM-DD HH:mm:ss')
'2017-05-11 13:23:58'

>>> local.humanize()
'an hour ago'

# 转为标准库对象
>>> a.date()
datetime.date(2017, 5, 7)

>>> a.time()
datetime.time(4, 38, 15, 447644)
```



## 总结

arrow是不是很智能很易用？如果以后你的Python项目需要处理时间，请果断抛弃标准库，arrow将拯救你无数脑细胞。

附上arrow官方文档，更多酷炫用法还是前往官网。

- http://arrow.readthedocs.io/en/latest/
