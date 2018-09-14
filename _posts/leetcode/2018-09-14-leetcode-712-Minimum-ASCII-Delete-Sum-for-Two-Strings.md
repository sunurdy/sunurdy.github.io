---
layout:     post
title:      712. Minimum ASCII Delete Sum for Two Strings
category: leetcode
description: DP
---

## Minimum ASCII Delete Sum for Two Strings
### description

Given two strings s1, s2, find the lowest ASCII sum of deleted characters to make two strings equal.

**Example 1**:

Output: 231
Explanation: Deleting "s" from "sea" adds the ASCII value of "s" (115) to the sum.
Deleting "t" from "eat" adds 116 to the sum.
At the end, both strings are equal, and 115 + 116 = 231 is the minimum sum possible to achieve this.



**Example 2**:

Output: 403
Explanation: Deleting "dee" from "delete" to turn the string into "let",
adds 100[d]+101[e]+101[e] to the sum.  Deleting "e" from "leet" adds 101[e] to the sum.
At the end, both strings are equal to "let", and the answer is 100+101+101+101 = 403.
If instead we turned both strings into "lee" or "eet", we would get answers of 433 or 417, which are higher.

**Note**:

0 < s1.length, s2.length <= 1000.
All elements of each string will have an ASCII value in [97, 122].

## 我的思路
动态规划

时间复杂度O(m*n)

空间复杂度O(m*n)

```python
class Solution(object):
    def minimumDeleteSum(self, s1, s2):
        """
        :type s1: str
        :type s2: str
        :rtype: int
        """
        
        l1, l2 = len(s1), len(s2)
        # init
        dp = [[0] * (l1 + 1) for _ in xrange(l2 + 1)]
        
        # init 
        for i in xrange(l1, 0, -1):
            dp[l2][i - 1] = dp[l2][i] + ord(s1[i - 1])
        
        # init    
        for i in xrange(l2, 0, -1):
            dp[i - 1][l1] = dp[i][l1] + ord(s2[i - 1])
        
        # dp
        for i in xrange(l1 - 1, -1, -1):
            for j in xrange(l2 - 1, -1, -1):
                
                if s1[i] == s2[j]:
                    dp[j][i] = dp[j + 1][i + 1]
                else:
                    dp[j][i] = min(dp[j][i + 1] + ord(s1[i]), dp[j + 1][i] + ord(s2[j]))

        return dp[0][0]
```



