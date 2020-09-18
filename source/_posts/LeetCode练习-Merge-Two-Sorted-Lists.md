---
title: LeetCode练习-Merge Two Sorted Lists
date: 2019-07-04 20:42:36
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表

---

### 题目:

[LeetCode 21.Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

### 思路:

基本思想就是新建一个链表, 然后从两个列表的头结点分别依次比较大小, 并将较小的插入新链表中.
实现1: 递归实现
实现2: 两个链表依次遍历插入新链表
<!--more-->

### 代码实现:

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        
        if (l2 == null) return l1;
        
        ListNode temp = null;
        if(l1.val > l2.val){
            temp = l2;
            temp.next = mergeTwoLists(l1, l2.next);
        }else{
            temp = l1;
            temp.next = mergeTwoLists(l1.next, l2);
        }
        return temp;
    }
}
```

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null || l2 == null) {
            return l1 == null ? l2 : l1;
        }
        ListNode support = new ListNode(0);
        ListNode temp = support;
        ListNode temp1 = l1;
        ListNode temp2 = l2;
        while (temp1 != null && temp2 != null) {
            if (temp1.val <= temp2.val) {
                temp.next = temp1;
                temp1 = temp1.next;
            }else {
                temp.next = temp2;
                temp2 = temp2.next;
            }
            temp = temp.next;
        }
        if(temp1 != null) {
            temp.next = temp1;
        }else {
            temp.next = temp2;
        }
        return support.next;
    }
}
```