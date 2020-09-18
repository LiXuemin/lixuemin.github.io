---
title: Remove Nth Node From End of List
date: 2019-06-24 22:30:42
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
---
### 题目

[Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

### 思路

思路一:

1. 单链表, 无法从尾结点往前遍历. 那么可以将倒数第N个结点转为第`L-N+1`个正数节点
2. 长度L可以提前遍历一次来求值
3. 移除第`L - N + 1`个节点

思路二:

1. 利用双指针
2. 先让两个指针相隔`N - 1`的位置
3. 然后双指针同步后移
4. 当第二个指针到了尾部, 第一个指针恰好在倒数第N个节点的前驱位置.
<!--more-->

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
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // return removeByTwoPass(head, n);
        return removeByTwoPointer(head, n);
    }
    
    //Method2: two pointer solution
    private ListNode removeByTwoPointer(ListNode head, int n) {
        ListNode support = new ListNode(0);
        support.next = head;
        
        //防止first.next.next删除节点时空指针错误
        ListNode first = support;
        ListNode second = support;
        int i = 0;
        while (i < n) {
            second = second.next;
            i++;
        }
        while (second != null && second.next != null) {
            second = second.next;
            first = first.next;
        }
        first.next = first.next.next;
        return support.next;
    }
    
    //Method1: get length, and remove (length -n + 1)th node
    private ListNode removeByTwoPass(ListNode head, int n) {
        ListNode support = new ListNode(0);
        support.next = head;
        int length = 1;
        ListNode iter = head;
        while(iter.next != null) {
            iter = iter.next;
            length++;
        }
        length -= n;
        //在头节点前加了一个哨兵节点
        iter = support;
        while(length > 0) {
            length--;
            iter = iter.next;
        }
        iter.next = iter.next.next;
        return support.next;
    }
}
```
