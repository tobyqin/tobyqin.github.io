---
title: LeetCode第二题 - 两数相加
categories: [Tech]
tags: [leetcode,algorithm]
date: 2019-02-12
---

LeetCode 备忘。

<!-- more -->

## 题目

给定两个非空的链表，分别代表两个正整数。链表中存储的数字和实际的位数刚好相反，要求将这两个数字相加并以链表的结构返回。

举例说明：

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

假设给定的数字没有0开头（链表末位数肯定不是0），并且任意相加数也不为0。

## 解法一

这道题目基本上就是模拟学生时代的加法，主要考虑遍历和进位的问题。

比较直接的做法就是：

1. 先遍历链表1，同时加上链表2对应数位的数字存入结果，暂时不考虑进位的问题。
2. 如果链表1遍历结束后，链表2还没结束就直接剩下的直接加入结果。
3. 遍历结果集，大于10的进位取余，直至结束。

且看代码：

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
         # 相加结果集
        r = []
        
        # 遍历链表1
        while l1:
            if l2: 
                # 加上链表2相同位数的数字，忽略进位
                r.append(l1.val + l2.val)
                l2 = l2.next
            else:
                # 如果链表2已经结束，直接补位链表1
                r.append(l1.val)
            
            # 处理下一位数
            l1 = l1.next
        
        # 如果链表2没结束，直接补位
        while l2:
            r.append(l2.val)
            l2 = l2.next
        
        head = ListNode(0) # 结果链表
        current = head
        carry =0 # 进位数
        length = len(r)
        
        #遍历结果集
        for i in range(length):
            current.val = r[i] + carry
            
            # 大于10需要进位
            carry = current.val // 10
            # 取余为当前位数
            current.val = current.val % 10
            
            # 只处理到倒数第2位，重置current
            if i < length - 1:
                current.next = ListNode(0)
                current = current.next
        
        # 如果末位数还有进位，补一下
        if carry:
            current.next = ListNode(carry)
                
        return head
```

提交看看，68ms，打败92%的玩家，还不错。

## 解法二

加法我们当然可以自己算啦，也可以让CPU给我们算啊。所以解法二是一种赖皮的做法，就是把两个链表变成真正的整数，然后相加，然后再转成链表，你猜猜速度是更快还是更慢？

```python
class Solution:
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        data_1 = "" # 字符串
        data_2 = "" # 字符串
        
        # 链表1转字符串
        while (l1 != None):
            data_1 += str(l1.val)
            l1 = l1.next

        # 链表2转字符串
        while (l2 != None):
            data_2 += str(l2.val)
            l2 = l2.next

        # 字符串翻转后转整数
        data_1 = int(data_1[::-1])
        data_2 = int(data_2[::-1])

        # 相加后再转成字符串然后翻转
        ans = str(data_1 + data_2)[::-1]
		
        # 字符串拆开存放到数组
        ret = []
        for i in range(len(ans)):
            ret.append(ListNode(int(ans[i])))

        # 遍历数组生成链表
        for i in range(len(ret) - 1):
            ret[i].next = ret[i+1]

        return ret[0]
```

提交看看，80ms，打败了52%的玩家！看来这个来回倒腾的过程挺费CPU的，但是不费脑子啊。

