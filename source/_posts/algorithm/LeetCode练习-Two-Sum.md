---
title: LeetCode练习-Two Sum
date: 2019-06-12 21:47:45
categories: 算法日常练习
tags:
- 数据结构与算法
---
###问题链接: [Two Sum](https://leetcode.com/problems/two-sum/)
<p>最先想到的暴力解法:</p>

```java
public int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target){
                return new int[]{i, j};
            }
        }
    }
    throw new IllegalArgumentException("no solution");
}
```
<!--more-->
但是这样的时间复杂度是O(n<sup>2</sup>),空间复杂度O(1). 从运行时间上来看是18ms,比38.86%的提交快, 可以看出时间不是很理想.<br/>
利用Map将时间复杂度减为O(n),代码如下:

```java
public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            map.put(nums[i], i);
        }
        for (int i = 0; i< nums.length; i++){
            int result = target - nums[i];
            if (map.containsKey(result) && map.get(result) != i){
                return new int[]{i, map.get(result)};
            }
        }
        throw new IllegalArgumentException("No Solution");
    }
```
