---
title: LeetCode 350. 两个数组的交集
date: 2020-06-20 09:59:01
categories: 算法日常练习
tags:
- 数据结构与算法
---

### 题目

[leetcode 350. Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/)

### 思路

1. 最原始的想法就是判断数组1的元素, 是否出现在数组2中. 
2. 但如果有重复的元素会导致结果错误, 所以要忽略掉已经判断属于交集的两个数组中的元素
3. 在这里借用map来协助计算, map结构: <元素, 出现次数>, 遍历数组1并存入map中
4. 遍历数组2, 判断是否在map中, 存在则value--
5. 这里并没有新建数组存储结果, 而是直接存到了数组2中, 因为移动计算过的数据不再需要

### 进阶问题

1. 若两个数组有序, 算法能否优化?
<!--more-->

### 代码实现

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
      //valid param
      if (nums1 == null || nums2 == null || nums1.length ==0 || nums2.length == 0) {
          return new int[]{};
      }
      //convert array1 to map
      Map<Integer, Integer> map = new HashMap<>();
      for(int i : nums1) {
          map.put(i, map.getOrDefault(i, 0) + 1);
      }
        
      //compare array2 and map
      int index = 0;
      for(int j : nums2) {
          if (map.containsKey(j) && map.get(j) > 0) {
              map.put(j, map.get(j) - 1);
              nums2[index] = j;
              index++;
          }
      }
           
      //return result
      return Arrays.copyOf(nums2, index);
    }
}
```
