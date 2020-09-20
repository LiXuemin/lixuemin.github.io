---
title: LeetCode 15. 三数之和
date: 2020-07-01 15:31:59
categories: 算法日常练习
tags:
- 数据结构与算法
---

### 题目

[LeetCode 15. 3Sum](https://leetcode.com/problems/3sum/submissions/)

### 思路

1. 采用固定一个数, 双指针寻找另外两个数
2. 排序后开始处理, 注意几个情况:
   1. nums[i] > 0, 代表后面的数都会大于0, 没必要计算了
   2. 跳过固定数值重复的情况
   3. 跳过left指针、right指针重复的情况
<!--more-->
### 代码实现

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> list = new ArrayList<>();
        if(nums == null || nums.length < 3) {
            return list;
        }

        Arrays.sort(nums);
        int sum = 0;
        for (int i = 0; i < nums.length - 2; i++) {
            int l = i + 1;
            int r = nums.length - 1;
            if (nums[i] > 0) {
                break;
            }
            
            if (i == 0 || nums[i] != nums[i - 1]){
                while (l < r) {
                    sum = nums[i] + nums[l] + nums[r];
                    if (sum == 0) {
                        list.add(Arrays.asList(nums[i], nums[l], nums[r]));
                        while (l < r && nums[l] == nums[l + 1]) {
                            l++;
                        }
                        while (l < r && nums[r] == nums[r - 1]) {
                            r--;
                        }
                        l++;
                        r--;
                    }else if (sum < 0) {
                        l++;
                    }else{
                        r--;
                    }
                }
            }
        }        
        return list;
    }
}
```