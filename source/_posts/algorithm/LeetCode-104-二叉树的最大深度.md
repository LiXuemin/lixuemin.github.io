---
title: LeetCode 104. 二叉树的最大深度
date: 2020-07-24 16:16:40
categories: 算法日常练习
tags:
- 数据结构与算法
- 二叉树
---

### 题目

[LeetCode 104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

### 思路

思路一: 广度优先搜索, 层层遍历
思路二: 递归算法, 比较最大值

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
    public int maxDepth(TreeNode root) {
        int depth = 0;
        if (root == null) {
            return depth;
        }
        List<TreeNode> line = new ArrayList<>();
        List<TreeNode> temp;
        line.add(root);
        while (!line.isEmpty()) {
            temp = new ArrayList<>();
            for (TreeNode node : line) {
                if (node.left != null) {
                    temp.add(node.left);
                }
                if (node.right != null) {
                    temp.add(node.right);
                }
            }
            line = temp;
            depth++;
        }
        return depth;
    }
}
```

递归解法:

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int leftDepth = maxDepth(root.left);
        int rightDepth = maxDepth(root.right);
        return Math.max(leftDepth, rightDepth) + 1;
    }
}
```