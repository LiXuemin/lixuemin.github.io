---
title: LeetCode 122. 买卖股票的最佳时机 ii
date: 2020-06-25 15:43:54
categories: 算法日常练习
tags:
- 数据结构与算法
- 数组
---

### 题目

[LeetCode 122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

### 思路

1.  要求尽可能多的完成交易
2. 要求不能同时多笔交易
3. 所以判断每笔交易能不能赚钱(即 arr[i + 1] - arr[i] > 0), 满足条件则发生交易即可

<!--more-->

### 代码实现

```java
class Solution {
    public int maxProfit(int[] prices) {
        //validate param
        if(prices == null || prices.length <= 1) {
            return 0;
        }
        
        int totalProfit = 0;
        int profit = 0;
        //calculate profit between every day, if profit > 0, means you can complete one transaction.
        for (int i = 0; i < prices.length - 1; i++) {
            profit = prices[i + 1] - prices[i];
            if (profit > 0) {
                totalProfit += profit;
            }
        }
        return totalProfit;
    }
}
```
