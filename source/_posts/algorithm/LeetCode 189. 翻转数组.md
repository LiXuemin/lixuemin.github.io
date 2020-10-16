---
title: LeetCode 189. 翻转数组
date: 2020-06-26 17:42:54
categories: 算法日常练习
tags:
- 数据结构与算法
- 数组
---

### 题目

[LeetCode 189. Rotate Array](https://leetcode.com/problems/rotate-array/)

### 思路

#### 实现1: 利用一个额外的数组

1. 目标: 将数组1中每一个元素, 移动到额外数组目标索引位置
2. 目标索引 = (现在位置 + 步数) % 长度
3. 最终将额外数组元素再复制回原数组
PS: 这里要注意 java中 `Arrays.copyOf()`, `System.arraycopy()`, `clone() (对象是深拷贝, 对数组是浅拷贝)` 几种浅拷贝

#### 实现2: 分析数组翻转后的元素排列, 使用位置互换
<!--more-->
1. 翻转所有元素
2. K要先取模, 因为翻转 n(数组长度)的倍数是跟原数组一样
3. 前K个翻转, 再将剩余翻转

### 代码实现

```java
class Solution {
    public void rotate(int[] nums, int k) {
        if(nums.length == 1 || k == 0) {
            return;
        }
        //rotateByExtraArray(nums, k);
        rotateByReplace(nums, k);
    }
    
    /**
    * rotate by extra array
    */
    private void rotateByExtraArray(int[] nums, int k) {
        int[] result = new int[nums.length];
        for (int i = 0; i < nums.length; i++) {
            result[(i + k) % nums.length] = nums[i];
        }
        
        for (int i = 0; i < nums.length; i++) {
            nums[i] = result[i];
        }
    }
    
    /**
    * rotate by replace
    */
    public void rotateByReplace(int[] nums, int k) {
        
        k = k % nums.length;

        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, k-1);
        reverse(nums, k, nums.length - 1);
    }
    
    //reverse items in [i,j]
    private void reverse(int[] nums, int i, int j){
        for(; i < j; i++,j--){
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }     
    }
}
```