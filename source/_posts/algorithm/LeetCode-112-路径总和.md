---
title: LeetCode 112. 路径总和
date: 2020-07-26 14:09:47
categories: 算法日常练习
tags:
- 数据结构与算法
- 二叉树
---

### 题目

[LeetCode 112. 路径总和](https://leetcode-cn.com/problems/path-sum/)

### 思路

这道题, 递归和广度优先都可以, 关键在于迭代的时候, 可以通过传递sum与节点值的`差值`来得到结果. 

当`差值`为`0`, 代表确实存在这样一条路径. 

如果使用额外的数据结构存储每条路径的和, 反而会极大增加代码复杂度.
<!--more-->
### 代码实现

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        return hasPath(root, sum);
    }

    private boolean hasPath(TreeNode node, int sum) {
        if (node == null) {
            return false;
        }
        int result = sum - node.val;
        if (result == 0 && node.left == null && node.right == null) {
            return true;
        }
        return hasPath(node.left, result) || hasPath(node.right, result);
    }
}
```
