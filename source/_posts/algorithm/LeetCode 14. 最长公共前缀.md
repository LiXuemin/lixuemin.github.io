---
title: LeetCode 14. 最长公共前缀
date: 2020-06-24 21:34:28
categories: 算法日常练习
tags:
- 数据结构与算法
---

### 题目

[LeetCode 14. Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

### 思路

1. index从0开始, 每个字符串取第index个元素, 判断是否相等, 相等则index递增继续计算, 直到不相等为止
2. 个人觉得问题比较简单, 但是各种边界条件必须考虑到位
   * 数组为空, 数组只有一个字符串, 数组包含""
   * 因为是求前缀, 当index == 0 的元素不相等, 代表不存在最长公共前缀, 可以直接返回 ""
   * 注意 str.charAt(index) 越界问题
<!--more-->
### 代码实现

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        // validate array is null
        if (strs == null || strs.length == 0) {
            return "";
        }
        // if array.length = 1, return array itself
        if (strs.length == 1) {
            return strs[0];
        }
        int minLen = getStrsMinlength(strs);
        // if array.contains(""), return ""
        if(minLen == 0) {
            return "";
        }
        int index = 0;
        StringBuilder sb = new StringBuilder();
        // foreach char in array[0]
        for (char c : strs[0].toCharArray()) {
            boolean flag = true;
            for(String str : strs) {
                if (c != str.charAt(index)) {
                    flag = false;
                }
            }
            // if first is not equal, direct return ""
            if(index == 0 && !flag) {
                return "";
            }
            if(flag) {
                sb.append(c);
            }
            if (++index == minLen) {
                break;
            }
        }
        return sb.toString();
    }
    
    //return the min length of string array
    public int getStrsMinlength(String[] strs) {
        int min = strs[0].length();
        for(String str : strs) {
            min = Math.min(str.length(), min);
        }
        return min;
    }
}
```
