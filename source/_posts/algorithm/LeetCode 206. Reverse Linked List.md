---
title: LeetCode 206. Reverse Linked List
date: 2020-06-18 21:37:04
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
---

### 题目:

 [LeetCode 206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

### 思路:

链表的问题说简单也简单, 但是手写代码时也挺容易出错的,主要考验对边界问题的处理.
<br>
单链表反转我觉得重点在于<b>头节点的后继指针指向<code>null</code></b>
<br>
其次循环中的逻辑就比较简单了:
<!--more-->
1. 保存临时后继结点
2. 把指针指向前驱结点
3. 指定当前结点为前驱, 后继结点为当前, 进行下一个结点的反转

### 代码实现

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
 class solution {
     public ListNode reverseList(ListNode head) {
         ListNode currNode = head;
         ListNode prevNode = null;
         ListNode nextTemp = null;
         while (currNode.next != null) {
             nextTemp = currNode.next;
             currNode.next = prevNode;
             prevNode = currNode;
             currNode = nextTemp;
         }
         return prev;
     }
 }
```