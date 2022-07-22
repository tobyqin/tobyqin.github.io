---
title: LeetCode第一题 - 两数之和
categories: [Tech]
tags: [leetcode,algorithm]
date: 2019-02-10
---

LeetCode 备忘。

<!-- more -->

## 题目

给定一个整数数组，找出和为指定值的两个元素的下标。举例说明：

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

假设数组中有且只有一种组合可以得到正确答案，并且相同元素不可以重复使用。

## 解法一

很容易想到暴力破解，只要做两层循环就可以搜索出答案。实现起来和冒泡排序类似。

```python
class Solution(object):
    def twoSum(self, nums, target):
        length = len(nums)
        for i in range(length):
            for j in range(i+1,length):
                if nums[i] + nums[j] == target:
                    return [i,j]
        
```

这个解法效率很低，运行2900ms，只打败了全球20%的玩家。

## 解法二

我们可以优化一下，用哈希表（字典）来保存数字的索引和值，这样搜索的复杂度就变成了O(1)，而遍历的复杂度是N。

```python
class Solution(object):
    def twoSum(self, nums, target):
        length = len(nums)
        # 建立index和value的反向索引
        d = {x:i for i,x in enumerate(nums)}
        
        for i in range(length):
            part = target - nums[i] # 取差值
            if part in d and not d[part]==i: # 在字典中搜索差值
                return [i,d[part]]
```

提交看疗效，20ms，打败全球100%玩家。

## 解法三

其实还可以优化一下，这个哈希表可以延迟建立，这样可以省掉建表时的那次遍历。但效果嘛，不一定是优化，且看代码。

```python
class Solution(object):
    def twoSum(self, nums, target):
        length = len(nums)
        d = {} # 空字典
        
        for i in range(length):
            part = target - nums[i]
            if part in d:  # 检索字典，有则直接返回
                return [i,d[part]]
            else: # 没找到，加入字典
                d[nums[i]]=i
```

提交，也是20ms，依然打败100%全球玩家。不过要注意一点，这种算法返回的下标都是反的，比如上面两种算法返回的是`[0,1]`，但是这里返回的就是`[1,0]`了。