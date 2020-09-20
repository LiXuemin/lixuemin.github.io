---
title: LeetCode 876. Middle of the Linked List
date: 2020-06-23 21:51:37
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
- 双指针
---
### 题目: 

[LeetCode 876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/)

### 思路:

刷到这个题的时候, 因为之前做过快慢指针[判断链表中是否存在环](https://blog.lixuemin.com/2019/06/20/LeetCode%E7%BB%83%E4%B9%A0-Linked-List-Cycle/)的问题.<br/>
最先想到的解法就是: 
<b>因为快指针的速度是慢指针的一倍, 所以当快指针走到尾结点的时候, 慢指针恰好走到中间.</b>
<!--more-->
<br>
需要注意的就是中间结点的判断:
1. 假如<code>fast.next == null</code>, 代表链表结点的个数是偶数, 此时<code>slow</code>处在中间位置偏左, 因为题目要求取偏右的, 所以返回<code>slow.next</code>
<br/> eg: [1, 2, 3, 4, 5, 6], 最终<code>fast = 6</code>,而<code>slow = 3</code>, 故需要返回<code>slow.next</code>即<code>4</code>
<br/>
2. 假如<code>fast.next != null && fast.next.next == null</code>, 代表链表结点数为奇数, 此时<code>fast</code>是尾结点的前驱结点, <code>slow</code>是中间结点的前驱结点, 故返回<code>slow.next</code>

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
    public ListNode middleNode(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode slow = head;
        ListNode fast = head.next;
        while (true) {
            if (fast.next == null || fast.next.next == null) {
                return slow.next;
            }
            slow = slow.next;
            fast = fast.next.next;
        }
        
    }
}
```