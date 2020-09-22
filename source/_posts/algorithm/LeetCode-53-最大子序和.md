---
title: LeetCode 53. 最大子序和
date: 2020-07-22 16:10:22
categories: 算法日常练习
tags:
- 数据结构与算法
- 动态规划
---

### 题目

[LeetCode 53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

### 思路

* 对数组进行遍历，当前最大连续子序列和为 sum，缓存结果为 ans
* 如果 sum > 0，则说明 sum 结果是好的, 累加当前遍历的值
* 如果 sum <= 0，则sum结果是坏的, 需要舍弃，则 sum 直接更新为当前遍历的值
* 每次比较 sum 和 ans的大小，将最大值置为ans，遍历结束返回结果
<!--more-->
### 代码实现

```java
class Solution {
    public int maxSubArray(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        if (nums.length == 1) {
            return nums[0];
        }

        int sum = 0;
        int ans = nums[0];
        
        for (int n : nums) {
            if (sum <= 0) {
                sum = n;
            }else {
                sum += n;
            }
            ans = Math.max(sum, ans);
        }
        return ans;
    }
}
```