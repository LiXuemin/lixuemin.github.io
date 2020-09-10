---
title: LeetCode练习-Merge Two Sorted Lists
date: 2019-06-23 20:42:36
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
- 合并两个有序链表
---
### 问题链接:[Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)
<p>基本思想就是新建一个链表, 然后从两个列表的头结点分别依次比较大小, 并将较小的插入新链表中.</p>
<!--more-->
代码如下:<br/>

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
