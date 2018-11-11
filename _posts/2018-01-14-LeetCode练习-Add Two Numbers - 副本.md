---
layout: post
title: LeetCode练习-Add Two Numbers
date:  2018-01-14 22:19:00
tags:
- Flask
- Python
categories: LeetCode
description: Add Two Numbers。
--- 
### Add Two Numbers
You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself. 
**Example**  
```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```  
### 解法1
该题目的第一个难点是怎样处理两个数字相加后进位的问题。通过 `/` 整除可以获得进位的值，用 `%` 余除可以得到当前位对应的结果。因此，该问题可以解决。
第二个问题是要解决输入的两个链表长度不一致的问题。短链表遍历完之后，将较长链表的值与进位值继续相加。当两个链表都遍历完之后，如果还有进位，则将该值初始为链表的节点，添加到链表末尾。
第三个问题要解决结果链表的第一个节点怎样设置。首先想到的是将两个链表的第一个元素取出来，单独相加，然后初始化第一个节点，这样是可以解决问题，但会导致冗余代码。如下所示：  

```Python
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
        if not l1:
            return l2
        if not l2:
            return l1
        sum = l1.val + l2.val
        flag = sum / 10
        value = sum % 10
        r = ListNode(value)
        head = r
        l1 = l1.next
        l2 = l2.next
        while l1 and l2:
            sum = l1.val + l2.val + flag
            flag = sum / 10
            value = sum % 10
            temp = ListNode(value)
            l1 = l1.next
            l2 = l2.next
            r.next = temp
            r = r.next
        if not l1:
            while l2:
                sum = l2.val + flag
                value = sum % 10
                flag = sum / 10
                l2 = l2.next
                temp = ListNode(value)
                r.next = temp
                r = r.next
        if not l2:
            while l1:
                sum = l1.val + flag
                value = sum % 10
                flag = sum / 10
                l1 = l1.next
                temp = ListNode(value)
                r.next = temp
                r = r.next
        if flag:
            r.next = ListNode(1)
        return head.next
``` 
其实结果链表的第一个节点可以用0进行初始化，这样就不用把连个链表的第一个值单独取出来。如下   

```python
class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        if not l1:
            return l2
        if not l2:
            return l1
        flag = 0
        head = ListNode(0)
        r = head
        while l1 and l2:
            sum = l1.val + l2.val + flag
            flag = sum / 10
            value = sum % 10
            temp = ListNode(value)
            l1 = l1.next
            l2 = l2.next
            r.next = temp
            r = r.next
        if not l1:
            while l2:
                sum = l2.val + flag
                value = sum % 10
                flag = sum / 10
                l2 = l2.next
                temp = ListNode(value)
                r.next = temp
                r = r.next
        if not l2:
            while l1:
                sum = l1.val + flag
                value = sum % 10
                flag = sum / 10
                l1 = l1.next
                temp = ListNode(value)
                r.next = temp
                r = r.next
        if flag:
            r.next = ListNode(1)
        return head.next
```
###解法二
在处理长度不一致的两个链表时，解法一还是有些冗余。下面的解法将相加后的结果放在作为加数的一个链表中，当剩余一个链表未遍历完，将其添加到结果链表中，继续和进位值相加。
```python
class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        result = l1
        if not l1:
            return l2
        if not l2:
            return l1
        pre = ListNode(0)
        pre.next = l1
        flag = 0
        while l1 and l2:
            sum = l1.val + l2.val + flag
            flag = sum / 10
            sum = sum % 10
            l1.val = sum
            pre = l1
            
            l1 = l1.next
            l2 = l2.next
        if l2:
            pre.next = l2
            l1 = l2
        while l1:
            sum = l1.val + flag
            flag = sum / 10
            sum = sum % 10
            l1.val = sum
            pre = l1
            l1 = l1.next
        if flag:
            l1 = ListNode(1)
            pre.next = l1
        return result
```