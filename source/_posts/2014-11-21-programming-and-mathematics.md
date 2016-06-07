---
layout: post
title: '编程与数学'
date: 2014-11-21 04:43
comments: true
categories: [Python, 函数, LeetCode, 链表循环起点]
---
#瞎扯
这几天抽空在做[leetcode](https://oj.leetcode.com/problems)，越做越有种工作越久数学退化越厉害的感觉，很多题目有点数学方面的思维都会更容易很多，这边博文就拿一道找链表循环起点的题目来分析下。
#Linked List Cycle II
##题目
Given a linked list, return the node where the cycle begins. If there is no cycle, return null.
Follow up:
Can you solve it without using extra space?
##解析
这道题目分两部分，第一部分是求是否存在循环，这很简单，设置两个点去遍历链表，其中一个点一次移动一个节点，另一个一次移动两个，因此他们肯定会相遇，只要求是否相遇就能知道是否存在循环。
那么，如何求循环的起点呢？先看下这个图
![](http://chuantu.biz/t/17/1416544742x1822610959.jpg)
图中从head到起点距离为x，相遇点里起点为y，周长为L，相遇时fast节点已经走了n次循环。
那么，设lSlow为相遇时慢节点走过的距离，lFast为相遇时快节点走过的距离
那么，lSlow = x+y ; lFast = x+y+n\*L;lFast = 2lSlow;
从上面几个算式可以推导出，x = n\*L - y
左边加1个L减1个L, x = (n-1)\*L + (L-y)
转换成中文就是“Head到循环起点的距离 等于 几个循环 + 相遇点到循环起点的距离 ”
那么，把slow点丢回Head处，两个点同时一次走一个点的前进，那么slow点走过x后就会在循环起点处与fast点相遇，这样就可以求出相遇点。
##代码
按上面思路，代码就可以这样写

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    # @param head, a ListNode
    # @return a boolean
    def detectCycle(self, head):
        if not head or not head.next: return None
        slow = fast = head
        while fast and fast.next:
            fast = fast.next.next
            slow = slow.next
            if fast == slow:
                slow = head
                while slow != fast:
                    fast = fast.next
                    slow = slow.next
                return slow
        return None
```
##总结
不要死脑筋思考逻辑问题，其实用高中的数学就能很好的解决该问题。通过这道题，也增加了一点以后写程序的思维方式吧。