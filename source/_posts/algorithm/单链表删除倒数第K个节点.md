---
title: 单链表删除倒数第K个节点
date: 2020-06-14 22:42:33
categories: 算法日常练习
tags:
- 数据结构与算法
- 链表
---

### 题目

实现一个函数, 可以删除单链表中倒数第K个节点.
要求:
如果链表长度为N, 时间复杂度达到O(N),额外空间复杂度达到O(1)

### 思路

假设让链表从头开始走到尾, 每移动一步就让K值减一,则当链表走到结尾时:

* K > 0, 代表K - N > 0, 则K > N,说明链表不存在倒数第K个节点, 不用调整链表
* K = 0, 代表K = N, 倒数第K个节点就是头节点, 删除头节点即可
* K < 0, 代表存在该节点, 单链表已经遍历过了, 那么怎么找到要删除的节点的前驱节点呢? (通过node.next = node.next.next来删除)
  1. 重新从头节点遍历, 每移动一步让K加一
  2. 当K等于0时, 移动停止, 移动到的节点就是要删除节点的前驱节点
<!--more-->
     解释:
     * 要删除的节点是倒数第K个, 也就是正数 N-K+1个, 那么它的前驱节点就是第N-K个
     * 第一次遍历后, K的值变为K-N, 第二次遍历加到0停止, 就是 0 - (K - N) = N - K
     * 魔鬼逻辑, 毫无漏洞...

### 代码实现

```Java
public class Node {
    public int value;
    public Node next;
    public Node(int data) {
        this.value = data;
    }
}

public Node removeLastKthNode (Node head, int lastKth) {
    if (head == null || lastKth < 1) {
        return head;
    }
    Node cur = head;
    while (cur != null) {
        lastKth--;
        cur = cur.next;
    }
    if (lastKth > 0 ) {
        return head;
    }

    if (lastKth == 0) {
        head = head.next;
        return head;
    }

    if (lastKth < 0) {
        cur = head;
        while (++lastKth != 0) {
            cur = cur.next;
        }
        cur.next = cur.next.next;
    }
    return head;
}

```
