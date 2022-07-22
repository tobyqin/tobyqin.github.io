---
title: Python：将数组中的元素导出到变量中 (unpacking)
tags: [python, python list, tips]
date: 2016-09-24 23:03:34
categories: Tech
---

Python 算法备忘。

<!-- more -->

## 问题描述

你需要将数组（list）或元组（tuple）中的一些元素导出到`N`个变量中。可能你并不希望通过遍历的方式。

## 解决方案

任何序列都可以通过简单的变量赋值方式将其元素分配到对应的变量中，唯一的要求就是变量的数量和结构需要和序列中的结构完全一致。

```python
p = (1, 2)
x, y = p
# x = 1
# y = 2

data = ['google', 100.1, (2016, 5, 31)]
name, price, date = data
# name = 'google'
# price = 100.1
# date = (2016, 5, 31)

name, price, (year, month, day) = data
# name = 'google'
# price = 100.1
# year = 2016
# month = 5
# day = 31
```

如果变量结构和元素结构不一致，你将会遇到以下错误：

```python
p = (1, 2)
x, y, z = p

Traceback (most recent call last):
  File "<pyshell#12>", line 1, in <module>
    x, y, z = p
ValueError: not enough values to unpack (expected 3, got 2)
```

其实这样的操作不限于元组和数组，在字符串中也是可以用的。Unpacking 支持大多数我们常见的序列，比如文件迭代，各种生成器等等。

```python
s = 'Hello'
a,b,c,d,e = s
# a = 'H'
# b = 'e'
```

如果导出过程中你想丢掉一些元素，其实 Python 并不支持这样的语法，不过你可以指定一些不常用的变量来达到你的目的。

```python
data = ['google', 100.1, (2016, 5, 31)]
name, _, (_,month,_) = data
# name = 'google'
# month = '5'
# other fileds will be discarded
```

本文主要是为了测试 Markdown 排版工具，微信公众号里的高亮代码一直是很麻烦的事情，让我头疼了很长一段时间。现在看起来不错哦。
