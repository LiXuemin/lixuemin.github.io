---
title: LeetCode练习-Remove Nth Node From End of List
date: 2019-06-24 22:30:42
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
---
### 问题链接:[Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
解法: 去除倒数第N个结点, 由于这是一个单链表, 无法从尾结点往前遍历. 那么可以将倒数第N个结点转为第<code>L-N+1</code>,那么思路就会清晰很多.
1. 遍历两次, 第一次获取<code>L</code>,即链表的长度
2. 第二次当遍历到L-N+1的位置时, 移除该结点即可
3. 注意头结点和尾结点的特殊处理
<!--more-->
4. 这里我们加入哨兵结点
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
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        int length  = 0;
        ListNode first = head;
        while (first != null) {
            length++;
            first = first.next;
        }
        length -= n;
        first = dummy;
        while (length > 0) {
            length--;
            first = first.next;
        }
        first.next = first.next.next;
        return dummy.next;
    }
}
```
