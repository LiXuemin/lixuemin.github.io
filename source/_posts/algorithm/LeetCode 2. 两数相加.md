---
title: LeetCode 2. 两数相加
date: 2020-07-21 09:50:01
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
---

### 题目

[LeetCode 2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/solution/)

### 思路

1. 两个逆序存储数值的链表, 要求返回数值相加结果并且逆序存储的链表.
2. 就两个链表一起遍历, 每一位相加:

    * 若小于10直接拼接到新链表中
    * 若大于10, 则个位数拼接到新链表, 十位数加入到下一个链表节点的计算中(实际就是进位)

其实就是两数相加, 链表的头节点就是个位, 依此类推. 
<!--more-->
### 代码实现

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
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode support = new ListNode();
        ListNode p = l1;
        ListNode q = l2;
        ListNode curr = support;
        int carry = 0;
        while (p != null || q != null) {
            int x = (p == null) ? 0 : p.val;
            int y = (q == null) ? 0 : q.val;
            int sum = carry + x + y;
            //余数
            carry = sum / 10;
            curr.next = new ListNode(sum % 10);
            curr = curr.next;
            if (p != null) {
                p = p.next;
            }
            if (q != null) {
                q = q.next;
            }
        }
        if (carry > 0) {
            curr.next = new ListNode(carry);
        }
        return support.next;
    }
}
```