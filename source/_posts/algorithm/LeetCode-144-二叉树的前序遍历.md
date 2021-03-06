---
title: LeetCode 144. 二叉树的前序遍历
date: 2020-07-23 22:06:02
categories: 算法日常练习
tags:
- 数据结构与算法
- 二叉树
---

### 题目

[LeetCode 144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

### 思路

前序遍历是按照`根-左-右`的顺序.

从根节点开始, 每次迭代弹出栈顶元素, 并将其孩子节点压入栈中, 先压入右孩子再压入左孩子.
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
    public List<Integer> preorderTraversal(TreeNode node) {
        List<Integer> list = new LinkedList<Integer>();
        if (node == null) {
            return list;
        }
	    Stack<TreeNode> stack = new Stack<>();
        stack.push(node);
        while (!stack.isEmpty()) {
            node = stack.pop();
            list.add(node.val);
            if (node.right != null){
                stack.push(node.right);
            }
            if (node.left != null) {
                stack.push(node.left);
            }
        }
        return list;
    }
}
```