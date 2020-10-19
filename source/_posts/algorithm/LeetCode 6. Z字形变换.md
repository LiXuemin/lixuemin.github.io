---
title: LeetCode 6. Z字形变换
date: 2020-07-02 16:45:22
categories: 算法日常练习
tags:
- 数据结构与算法
---

### 题目

[LeetCode 6. ZigZag Conversion](https://leetcode.com/problems/zigzag-conversion/submissions/)

### 思路

1. 最开始想的就是一个二维数组, 它把原字符串, 按照规律放入固定位置, 最后再拼接
2. 那么又分为两个问题
   * 二维数组要考虑越界和合并判断问题
   * 这个Z(明明是N!)的规律是什么
3. 关于Z的规律只能说, 在纸上画一下2行,3行的情况, 发现规律是`2n-2`个字母为一个循环周期, 再拿4行验证一下
4. 另外选用了字符串来替代一维数组, 直接拼接保存该行
   <!--more-->

### 代码实现

```java
class Solution {
    public String convert(String s, int numRows) {
        if (s == null || s.length() <= 0 || numRows <= 1) {
            return s;
        }
        String[] arr = new String[numRows];
        Arrays.fill(arr,"");
        int cycle = 2 * numRows - 2;
        int position = 0;
        int mod = 0;
        for (char c : s.toCharArray()) {
            mod = position % cycle;
            if (mod < numRows) {
                arr[mod] += c;
            }else{
                arr[cycle - mod] += c;
            }
            position++;
        }
        StringBuilder re = new StringBuilder();
        for(String line : arr) {
            re.append(line);
        }
        return re.toString();
    }
}
```