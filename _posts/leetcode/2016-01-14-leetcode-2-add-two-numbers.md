---
layout:     post
title:      2. Add Two Numbers
category: leetcode
description: Add Two Numbers
---

## Add Two Numbers
### description

You are given two linked lists representing two non-negative numbers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)

Output: 7 -> 0 -> 8

```
class ListNode(object):
    def __init__(self, x):
        self.val = x
        self.next = None
```

## 我的思路
### 1. 顺序执行 
```
def addTwoNumbers(l1, l2):
    """
    res = ListNode(0)
    start = res
    flag = 0
    while(l1 or l2):
        val1 = l1.val if l1 else 0
        val2 = l2.val if l2 else 0
        num = val1 + val2 + flag
        (num, flag) = (num, 0) if num < 10 else (num-10, 1)
        tmp = ListNode(num)
        res.next = tmp
        res = tmp
        l1 = l1.next if l1 else l1
        l2 = l2.next if l2 else l2
    if flag == 1:
        tmp = ListNode(1)
        res.next = tmp
    return start.next
```
不细心会出很多错误 感觉写得不好

一年前写的代码：

```
class Solution {
public:
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
        if(!l1) return l2;
        if(!l2) return l1;
        
        ListNode* head=new ListNode(0);
        ListNode* start=head;
        int sum=0;
        while(l1 || l2)
        {
            sum/=10;
            if(l1)
            {
                sum+=l1->val;
                l1=l1->next;
            }
             if(l2)
            {
                sum+=l2->val;
                l2=l2->next;
            }
            
            start->next=new ListNode(sum%10);
            start=start->next;
        }
        if(sum/10) start->next=new ListNode(1);
     
        return head->next;
    }
};
```
发现一年时间过去了，还是没进步。
### 2. 递归
略

## Better Answer
```
def addTwoNumbers(self, l1, l2):
    carry = 0
    root = n = ListNode(0)
    while l1 or l2 or carry:
        v1 = v2 = 0
        if l1:
            v1 = l1.val
            l1 = l1.next
        if l2:
            v2 = l2.val
            l2 = l2.next
        carry, val = divmod(v1+v2+carry, 10)
        n.next = ListNode(val)
        n = n.next
    return root.next
```

比我的答案清晰明了，主要点有：

* 统一处理，顺序执行，就不用考虑复杂的条件
* n.next = ListNode(0)


