---
layout:     post
title:      26. Remove Duplicates from Sorted Array
category: leetcode
description: Remove Duplicates from Sorted Array

---

## Remove Duplicates from Sorted Array
### description
<div class="question-content">
              <p></p><p>
Given a sorted array, remove the duplicates in place such that each element appear only <i>once</i> and return the new length.</p>

<p>
Do not allocate extra space for another array, you must do this in place with constant memory.
</p>

<p>
For example,<br>
Given input array <i>nums</i> = <code>[1,1,2]</code>,
</p>
<p>
Your function should return length = <code>2</code>, with the first two elements of <i>nums</i> being <code>1</code> and <code>2</code> respectively. It doesn't matter what you leave beyond the new length.
</p>
</div>

## 思路
### 1

```
int removeDuplicates(vector<int>& nums) {
    int len=nums.size();
    if (len<2) return len;
    int count=0;
    for(int i=1; i<len; i++)
    {
        if(nums[count]!=nums[i])
            nums[++count] = nums[i];         
    }
    return count+1;
}
```

```
    def removeDuplicates(self, A):
        if A == []:
            return 0
        count = 0;
        for i in range(1,len(A)):
            if A[i] != A[i-1]:
                count += 1
                A[count] = A[i]
        return count+1
```

### 2
```
int count = 0;
for(int i = 1; i < n; i++){
    if(A[i] == A[i-1]) count++;
    else A[i-count] = A[i];
}
return n-count;
```


