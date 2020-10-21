---
title: Flink-2. 流计算API窗口
date: 2020-09-12 22:21:49
categories: 流计算
tags: 
- Flink
- 窗口函数
---

## Window

窗口是另一类算子， 是DataStream的逻辑边界， 在第一个元素到达后被创建， 在生命周期结束后被销毁。

应用程序可以定义开窗机制， 触发器， 迟到生存期， 窗口聚合函数和清除器。

窗口分两大类， 即Keyed Window和 Non-Keyed Window。

## 窗口分类

### 滚动窗口

滚动窗口的时间长度是固定的， 且不同时间区间的窗口不会重叠， 可根据事件时间和处理时间定义。

### 滑动窗口

滑动窗口按照滑动步长将时间拆分成固定长度的窗口， 当滑动步长小于窗口长度时，相邻窗口间会重叠。

### 会话窗口

根据相邻元素之间的时间间隔确定会话窗口的边界， 分为固定时间间隔和动态时间间隔。

### 全局窗口

将相同KEY的元素聚合在一起，但是这种窗口没有起点也没有终点，因此必须自定义触发器、

### 增量式计算

拆分窗口的目的是将指定时间区间内的所有元素当成一个有界数据集， 以分析这个数据集的整体特征。

那么从实现上来看，等待所有元素都被收集后再进行计算是最简单的， 但是毫无疑问会在最后的时间里疯狂占用CPU。

如果我们采用增量式计算的设计，即数据进入引擎立刻被处理， 这样是可以提升整体计算性能。

这样的好处是让计算任务平均在每个时间点上， 不会出现某个时刻突然大量计算的问题，减轻最后的计算压力。当然也增加数据处理引擎架构设计的复杂度。

## 触发器

触发器原型中包括4类触发机制， 基于事件驱动。

（1）onElement: 每收到一个元素调用一次该方法。

（2）onProcessingTime: 根据注册的处理时间定时器触发。

（3）onEventTime: 根据注册的事件时间定时器触发。

（4）onMerge: 两个窗口合并时触发。

另外还提供了资源清除接口clear().

## 清除器

在触发器触发后， 窗口函数执行前或执行后清除窗口内元素，有以下两个方法：

1. 触发器被触发后， 窗口函数执行前， 清除窗口内元素

```java
void evictBefore(Iterable<TimestampdValue<T>> elements, int size,  W window, EvictorContext evictorContext);
```

2. 触发器被触发后，窗口函数执行后，清除窗口内元素

```java
void evictAfter(Iterable<TimestampdValue<T>> elements, int size,  W window, EvictorContext evictorContext);
```

提供了三种内置清除器： CountEvictor, DeltaEvictor, TimeEvictor

## 迟到生存期

Flink默认的迟到生存期为0， 即事件时间窗口在水印到来后结束， 无需考虑事件迟到的情况。

```scala
val input: DataStream[T] = ...

input
    .keyBy(...)
    .window(...)
    .allowedLateness(Time.seconds(10))
    ...
```