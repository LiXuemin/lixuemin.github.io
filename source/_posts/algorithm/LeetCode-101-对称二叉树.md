---
title: LeetCode 101. 对称二叉树
date: 2020-07-25 13:31:35
categories: 算法日常练习
tags:
- 数据结构与算法
- 二叉树
---

### 题目

[LeetCode 101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

### 思路

思路一: 广度优先搜索

1. 分成两个链表, 分别按顺序存储根节点的左右子树的每一层节点
2. 两个链表进行比较, 一个顺序, 一个逆序取出元素, 进行比较
3. 要注意不完全二叉树的null值

思路二: 递归

1. 递归过程:

* 判断当前两个节点的值是否相等
* 判断左子树的右节点, 与右子树的左节点, 是否对称
* 判断左子树的左节点, 与右子树的右节点, 是否堆成

2.递归终止条件:

* 两个节点都为`null` 返回`true`
* 只有一个节点为`null`返回`false`

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
    public boolean isSymmetric(TreeNode root) {
        if (root == null || (root.left == null && root.right == null)) {
            return true;
        }
        LinkedList<TreeNode> leftLine = new LinkedList<>();
        LinkedList<TreeNode> rightLine = new LinkedList<>();
        leftLine.add(root.left);
        rightLine.add(root.right);
        while (!leftLine.isEmpty() || !rightLine.isEmpty()) {
            if (!isTwoListEqual(leftLine, rightLine)) {
                return false;
            }
            leftLine = getChildren(leftLine);
            rightLine = getChildren(rightLine);
        }
        return true;
    }

    private boolean isTwoListEqual(LinkedList<TreeNode> left, LinkedList<TreeNode> right) {
        if (left.size() != right.size()) {
            return false;
        }
        TreeNode leftNode = null;
        TreeNode rightNode = null;
        for (int i = 0; i < left.size(); i++){
            leftNode = left.get(i);
            rightNode = right.get(left.size() - i - 1);
            if (leftNode == null && rightNode == null) {
                continue;
            }
            if (leftNode == null || rightNode == null) {
                return false;
            }
            if (leftNode.val != rightNode.val) {
                return false;
            }
        }
        return true;
    }

    private LinkedList<TreeNode> getChildren(LinkedList<TreeNode> list) {
        LinkedList<TreeNode> temp = new LinkedList<>();
        for (TreeNode node : list) {
            if (node == null){
                continue;
            }
            temp.add(node.left);
            temp.add(node.right);
        }
        return temp;
    }
}
```

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
    public boolean isSymmetric(TreeNode root) {
        if (root == null) {
            return true;
        }
        return isSymmetric(root.left, root.right);
    }
    public boolean isSymmetric(TreeNode leftTree, TreeNode rightTree) {
        if (leftTree == null && rightTree == null) {
            return true;
        }
        if (leftTree == null || rightTree == null) {
            return false;
        }
        return (leftTree.val == rightTree.val)
        && isSymmetric(leftTree.left, rightTree.right)
        && isSymmetric(leftTree.right, rightTree.left);
    }
}
```
