---
title: 设计一个有getMin功能的栈
date: 2020-06-07 21:05:00
categories: 算法日常练习
tags:
- 数据结构与算法
- 栈
---

### 题目

实现一个特殊的栈, 在实现栈的基本功能的基础上, 再实现返回栈中最小元素的操作.

### 要求

1. pop, push, getMin操作的时间复杂度都是O(1)
2. 设计的栈类型可以使用现有的栈结构

### 思路

先考虑下getMin的特殊情况

1. 应该有一个数据结构能存储,并在入栈和出栈时都要维护当前最小值
2. 可以用另一个栈来存储当前的最小值.
<!--more-->
### 代码实现

两种方案都用了stackMin栈来保存stackData中每一步的最小值.

共同点: 空间复杂度都是O(n), 时间复杂度都是O(1).

区别: 最小栈在方案二中, 冗余存储了每一步的最小值.优点在于出栈无需额外判断, 缺点是入栈时做额外判断.

```java
public class MyStack1 {
    private Stack<Integer> stackData;
    private Stack<Integer> stackMin;

    public MyStack1() {
      this.stackData = new Stack<Integer>();
      this.stackMin = new Stack<Integer>();
    }

    public void push(int newNum) {
      if (this.stackMin.isEmpty()) {
        this.stackMin.push(newNum);
      } else if (newNum <= this.getMin()) {
        this.stackMin.push(newNum);
      }
      this.stackData.push(newNum);
    }

    public int pop() {
      if (this.stackData.isEmpty()) {
        throw new RuntimeException("Your stack is empty.");
      }
      int value = this.stackData.pop();
      if (value == this.getMin()) {
        this.stackMin.pop();
      }
      return value;
    }

    public int getMin() {
      if (this.stackData.isEmpty()) {
        throw new RuntimeException("Your stack is empty.");
      }
      return this.stackMin.peek();
    }
  }
```

```java
public class MyStack2 {
    private Stack<Integer> stackData;
    private Stack<Integer> stackMin;

    public MyStack2() {
      this.stackData = new Stack<Integer>();
      this.stackMin = new Stack<Integer>();
    }

    public void push(int newNum) {
      if (this.stackMin.isEmpty()) {
        this.stackMin.push(newNum);
      }else if (newNum < this.getMin()) {
        this.stackMin.push(newNum);
      }else {
        this.stackMin.push(this.getMin());
      }
      this.stackData.push(newNum);
    }

    public int pop() {
      if (this.stackData.isEmpty()) {
        throw new RuntimeException("Your stack is empty");
      }
      this.stackMin.pop();
      return this.stackData.pop();
    }

    public int getMin() {
      if (this.stackMin.isEmpty()) {
        throw new RuntimeException("Your stack is empty");
      }
      return this.stackMin.peek();
    }
  }
```
