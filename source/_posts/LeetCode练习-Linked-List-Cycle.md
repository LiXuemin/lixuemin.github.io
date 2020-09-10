---
title: LeetCode练习-Linked List Cycle
date: 2019-06-20 22:35:39
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
---
### 问题链接:[Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)

最先想到的是遍历链表, 并将结点的内存地址存在<code>HashSet</code>中.如果当前结点的内存地址已经存在,则表示存在环,直接返回<code>true</code>.
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
            if (set.contains(head)) {
                return true;
            }else {
                set.add(head);
            }
            head = head.next;
        }
        return false;
    }
}
```
时间复杂度O(n)<br>
空间复杂度O(n)
<br>
后来看了解析, 原来还有快慢指针的解法
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