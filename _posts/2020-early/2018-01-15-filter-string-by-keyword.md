---
title: 【问题】使用Python将字符串过滤并保留关键字
categories: [Quiz]
tags: [python,quiz]
date: 2018-01-15
---

Python 算法备忘。

<!-- more -->

## 问题描述

给出一个关键词列表:

```
keys = ['aaa','bbb','ccc']
```

给出一个字符串 `str`，如果字符串中有包含列表 `keys` 中的子串，则过滤并保留下来，其余部分则删除。请问可以如何实现？

例子:

```
str = 'aaaawtf he heheccc'
```

过滤后变成:

```
'aaa ccc'
```

例子二：

```
str = 'aaabbb/&edfg cccaaa'
```

过滤后变成：

```
'aaabbb cccaaa'
```

要求尽可能保留原字符串的相对位置信息，比如aaa和bbb是连在一块的，就连在一块被保留下来。

## 实现思路

解决这个问题可以分两步走，第一步，找出关键字位置并记录；第二步，重新组合拼接。

```python
str = 'aaabbb/&edfg cccaaa'
keys = ['aaa', 'bbb', 'ccc']

found = {k: [] for k in keys}
total_length = len(str)

# 按key依次遍历字符串，保存出现的位置
for key in keys:
    length, i = len(key), 0

    while i + length <= total_length:
        s = str[i:i + length]

        if s == key:
            found[key].append(i)
            i += length
        else:
            i += 1

print(found)
# {'aaa': [0, 16], 'bbb': [3], 'ccc': [13]}

result, next_match_index = '', -1

# 重新组合，如果坐标重叠连接符为空，否则为空格
for i in range(total_length):
    for k, v in found.items():
        if i in v:
            split = '' if next_match_index == i else ' '

            result = result + split + k
            next_match_index = i + len(k)

print(result)
# aaabbb cccaaa
```



## 改进方案

其实可以考虑以上两步可以合并一起做掉，不过代码就相对没那么好理解了。

```python
str = 'aaaabbb/&edfg cccaaa'
keys = ['aaa', 'bbb', 'ccc']

total_length = len(str)
result, next_match_index, skip = '', -1, 0

for i in range(total_length):

    if skip:
        skip -= 1
        continue

    for key in keys:  # 查找当前位置是否有match的key
        length = len(key)

        if i + length <= total_length:  # 确保index不越界
            s = str[i:i + length]

            if s == key:  # 如果有match的key，添加到结果
                split = '' if next_match_index == i else ' '
                result = result + split + key

                next_match_index = i + length #预测相邻key的位置
                skip = length - 1 #需要跳过当前key后匹配下一个key
                break # 已经找到匹配key，可以结束keys的遍历

print(result)
```

