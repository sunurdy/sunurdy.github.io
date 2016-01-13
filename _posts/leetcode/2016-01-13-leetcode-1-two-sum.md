---
layout:     post
title:      1. Two Sum
category: leetcode
description: Two Sum
---

## Two Sum
### description

Given an array of integers, find two numbers such that they add up to a specific target number.
The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. 
Please note that your returned answers (both index1 and index2) are not zero-based.
You may assume that each input would have exactly one solution.

## 我的思路
### 1. O(N^2) 
两个for循环 略
### 2. 
先排序，再头尾两边往中间找。nlogn + n
有问题，只能找到对应的数字。还需要在原来的list中找到相应的位置。
不直观，很复杂。

## O(n) Answer
```
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        d={}
        for i,v in enumerate(nums):
            if target-v in d:
                return [d[target-v],i+1]
            d[v]=i+1
```



