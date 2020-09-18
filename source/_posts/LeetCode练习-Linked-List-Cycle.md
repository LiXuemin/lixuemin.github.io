---
title: Linked List Cycle
date: 2020-07-20 22:35:39
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
  
---

### 题目:

[Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

### 思路:

1. 最先想到的是遍历链表, 并将结点的内存地址存在`HashSet`中.如果当前结点的内存地址已经存在,则表示存在环,直接返回`true`
2. 还可以用双指针(快慢指针)来解决, 如果两个指针相遇(套圈), 则代表存在环.
<!--more-->

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }
        Set<ListNode> set = new HashSet<>();
        while (head != null) {
            if (set.contains(head)){
                return true;
            }
            set.add(head);
            head = head.next;
        }
        return false;
    }
}
```

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }
        ListNode slow = head;
        ListNode fast = head.next;
        while (slow != fast) {
            if (fast == null || fast.next == null) {
                return false;
            }
            slow = slow.next;
            fast = fast.next.next;
        }
        return true;
    }
}
```