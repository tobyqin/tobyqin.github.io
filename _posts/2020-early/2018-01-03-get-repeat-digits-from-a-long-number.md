---
title: 【问题】从一长串数字中找到重复多次的三个数字
categories: [Quiz]
tags: [quiz,python]
date: 2018-01-03
---

Python 算法备忘。

<!-- more -->

## 问题描述

> https://stackoverflow.com/questions/47581326/given-a-string-of-a-million-numbers-return-all-repeating-3-digit-numbers

假设给定一个很长的数字，比如PI精确到100万位，找到其中重复出现相邻三个数字。比如给定的数字是1233223332321234323123，那么结果应该是：

```
123 repeat 3 times
233 repeat 2 times
323 repeat 2 times
332 repeat 2 times
```

## 解决思路

如果要统计相邻三个数字的重复次数，那么必然需要将其所有可能都列出来，通过Python的切片我们很容易实现：

```Python
number = '1233223332321234323123'
split = [number[position:position + 3] for position in range(len(number) - 2)]

print(split)
# ['123', '233', '332', '322', '223', '233', '333', '332', '323', '232', '321', '212', '123', '234', '343', '432', '323', '231', '312', '123']
```

接下来我们需要统计切好的序列里各个数字出现的次数，因为需要处理是3位数字，可以考虑新建一个长度为1000的空序列，如果数字出现就在对应位置加一，达到统计的目的。

```python
seq = [0] * 1000

for x in split:
    seq[int(x)] += 1
```

最后我们只要把新序列里统计值大于1的打印出来即可。

```python
for i in range(1000):
    if seq[i] > 1:
        print('{} repeat {} times'.format(i, seq[i]))

# 123 repeat 3 times
# 233 repeat 2 times
# 323 repeat 2 times
# 332 repeat 2 times
```

我们可以用更加优雅的方式来呈现以上算法，简洁但不简单。

```python
seq = [0] * 1000

for val in [int(number[pos:pos+3]) for pos in range(len(number) - 2)]:
    seq[val] += 1

print ([(num, seq[num]) for num in range(1000) if seq[num] > 1])
```

以上便是Stack Overflow上原题的最佳答案。

## 拓展思考

如果这个问题给定的不是数字，而是字符串比如abccdbadfdaabc，依然是要找到相邻的3个重复字母，你有没有好办法？