---
title: Flink 0. 流处理基本概念
date: 2020-08-26 21:57:26
categories: 流计算
tags: 
- Flink
---

1. 无界流和有界流

    无界流: 持续生成的, 本质上无限的数据集合. 

    有界流: 有限的数据集合.

2. 延迟和吞吐量
3. 时间语义
    - Processing Time, 流计算引擎处理事件的时间
    - Event Time, 事件实际发生的时间
    - Watermarks，水印
    - Window    
      * 滚动窗口
      * 滑动窗口
      * 会话窗口
    - Trigger