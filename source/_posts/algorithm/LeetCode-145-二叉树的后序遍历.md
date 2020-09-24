---
title: LeetCode 145. 二叉树的后序遍历
date: 2020-07-23 22:06:49
categories: 算法日常练习
tags:
- 数据结构与算法
- 二叉树
---

### 题目

[LeetCode 145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

### 思路

后序遍历是按照`左-右-根`的顺序.

按照`根-左-右`的顺序将节点入栈, 出栈顺序则是后序遍历的值.
<!--more-->
### 代码实现

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        LinkedList<Integer> list = new LinkedList<>();
        if(root == null) {
            return list;
        }
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);
        while (!stack.isEmpty()) {
            root = stack.pop();
            list.addFirst(root.val);
            if (root.left != null) {
                stack.add(root.left);
            }
            if (root.right != null) {
                stack.add(root.right);
            }
        }
        return list;
    }
}
```
