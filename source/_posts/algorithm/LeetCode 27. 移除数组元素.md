---
title: LeetCode 27. 移除数组元素
date: 2020-6-28 20:45:19
categories: 算法日常练习
tags:
- 数据结构与算法
---

### 题目

[LeetCode 27. Remove Element](https://leetcode.com/problems/remove-element/)

### 思路

这道题目, 把题意读明白就好做了.
预定义的方法中要求返回`int类型长度len`, 但代码提交时, 却是按照 `nums中[0, len)的区间输出`判断的.
所以最终需要注意的是:

1. O(1) 空间复杂度
2. nums中跟val不等的元素, 要往前移
<!--more-->
### 代码实现

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        
        int i = 0;
        int j = 0;
        while (i < nums.length) {
            if (nums[i] == val) {
                i++;
                continue;
            }else{
                nums[j] = nums[i];
                i++;
                j++;
            }
        }
        return j;
    }
}
```