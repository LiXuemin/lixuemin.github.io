---
title: 记一次生产环境OOM问题
date: 2021-06-30 21:55:32
categories: 踩坑实战
tags:
- JVM
- OOM
---

### 使用命令及工具简介

* `top`, linux自带，查看当前最占资源的进程，按`SHIFT + M`可以重排序

* `jmap`，jdk自带堆内存工具

* `jstat`

* `arthas`

* `visualvm`

* `jprofiler`

### 排查思路及过程

1. 从整体到部分，逐步确认问题所在

**监控**看到的问题往往是最宏观的，所以首先可以从监控看起。通过监控面板， 查看主机或容器CPU、内存、连接数， 查看数据库连接数等信息。

后来看到机器的**CPU使用率100%，内存使用率90%**， 那么大概率是程序问题。

2. 


### 排查过程

1. arthas排查
发现占用CPU较高的全是GC线程
2. jmap -heap pid查看堆内存
发现老年代数据无法GC掉，占用20G+内存，使用率99%
3. jstat- gcutil pid 1000 10查看GC情况
发现每分钟一次FullGC，且无法GC掉
此时可以确定是程序问题，怀疑某处内存泄漏，导致无法GC清除数据
4. jmap -heap:live pid 查看堆内存中占用最多的类
发现几乎都是QuizExaminne类
5. heapdump下载堆转储文件，进一步分析堆内存中内容，排查问题
