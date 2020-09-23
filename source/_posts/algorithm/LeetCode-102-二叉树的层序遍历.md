---
title: LeetCode 102. 二叉树的层序遍历
date: 2020-07-23 22:07:05
categories: 算法日常练习
tags:
- 数据结构与算法
- 二叉树
---

### 题目

[LeetCode 102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

### 思路

这个实现还是挺简单的, 就是注意下每层迭代的时候, 数组要先清空.
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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> list = new ArrayList<List<Integer>>();
        if (root == null) {
            return list;
        }
        List<TreeNode> nodeList = new LinkedList<>();
        List<TreeNode> temp;
        List<Integer> line;
        nodeList.add(root);
        while(!nodeList.isEmpty()) {
            temp = new LinkedList<>();
            line = new LinkedList<>();
            for (TreeNode node : nodeList) {
                line.add(node.val);
                if (node.left != null) {
                    temp.add(node.left);
                }
                if (node.right != null) {
                    temp.add(node.right);
                }
            }
            nodeList = temp;
            list.add(line);
        }
        return list;
    }
}
```
