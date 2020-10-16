---
title: LeetCode 977.有序数组的平方
date: 2020-08-01 17:07:01
categories: 算法日常练习
tags:
- 数据结构与算法
- 双指针
---

### 题目

[LeetCode 977. 有序数组的平方](https://leetcode-cn.com/problems/squares-of-a-sorted-array/)

### 思路

1. 最简单的思路是首先遍历数组, 每个值求平方, 然后排序并返回

时间复杂度 = 遍历一次数组O(n) + 调用一次`Arrays.sort()` 快排O(n*logn)
即O(n + n* logn) = O(n * logn)

2. 另外仔细审题, 提示*已经按非递减顺序排序*, 所以说其实数据是由小到大排序的.
那么可以得出:

* 如果数组中没有负数, 那么只需要按顺序求平方并返回结果即可.
* 负数部分由小到大, 说明它们的绝对值是按*由大到小*排序的.
<!--more-->
那么用两个指针分别从前后开始, 比较绝对值大小, 最大的那个元素求平方放到数组最末位, 便能得到结果.

### 代码实现

遍历+排序

```java
class Solution {
    public int[] sortedSquares(int[] A) {
        int[] r = new int[A.length];
        int index = 0;
        for(int i : A) {
            r[index] = i * i;
            index++;
        }
        Arrays.sort(r);
        return r;
    }
}
```

双指针解法

```java
class Solution {
    public int[] sortedSquares(int[] A) {
        int[] r = new int[A.length];
        int left = 0;
        int right = A.length - 1;
        int index = A.length -1;
        while (index >= 0){
            if (Math.abs(A[left]) > Math.abs(A[right])) {
                r[index--] = A[left] * A[left];
                left++;
            } else {
                r[index--] = A[right] * A[right];
                right--;
            }
        }
        return r;
    }
}
```
