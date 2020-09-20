---
title: LeetCode 66. 加一
date: 2020-06-29 21:52:44
categories: 算法日常练习
tags:
- 数据结构与算法
---

### 题目

[LeetCode 66. Plus One](https://leetcode.com/problems/plus-one/)

### 思路

难度在于要考虑到所有情况:

1. 末位 ≠ 9, 直接++, 返回数组
2. 末位 == 9, 且存在某位 ≠ 9, 在该位 ++ , 返回数组
3. 末位 == 9 , 前面所有位都是9, 新建一个数组, 首位置为1, 返回
<!--more-->
### 代码实现

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int last = digits.length - 1;
        
        //最简单的情况,末位 != 9, 加1, 返回数组
        if (digits[last] != 9) {
          digits[last] = digits[last] + 1;   
          return digits;
        }
        
        for (int i = last; i >= 0; i--) {
            //当前位 != 9, 加1, 返回数组
            if (digits[i] != 9) {
                digits[i] = digits[i] + 1;
                return digits;
            }
            
            digits[i] = 0;
            //首位 == 9, 要新建数组, 首位进1, 其它位置为0, 返回
            if (i == 0) {
                int[] result = new int[last + 2];
                result[0] = 1;
                return result;
            }
        }
        return digits;
    }
}
```
