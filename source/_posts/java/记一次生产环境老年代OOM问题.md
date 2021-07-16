---
title: 记一次生产环境OOM问题
date: 2021-06-30 21:55:32
categories: 踩坑实战
tags:
- JVM
- OOM
---

### 使用命令及工具简介

* `top`, linux自带，查看当前最占资源的进程

* `jmap`，jdk自带堆内存工具

* `jstat`，查看GC情况

* `arthas`，阿里开源JVM性能分析工具
<!--more-->

* `visualvm`，开源JVM分析工具

* `jprofiler`，商业JVM分析工具

### 排查思路及过程

从整体到部分，逐步确认问题所在

**监控**看到的问题往往是最宏观的，所以首先可以从监控看起。通过监控面板， 查看主机或容器CPU、内存、连接数， 查看数据库连接数等信息。

此时看到机器的**CPU使用率100%，内存使用率90%**，其它指标比较正常，那么大概率是程序问题。


### 排查过程

最开始，业务开发希望协助使用arthas帮助寻找到底是什么方法比较慢。

1. arthas查看全局情况

![arthas dashboard](/images/oom/jvm-arthas-dashboard.png)

发现占用CPU较高的全是GC线程，此时就应该进一步查看GC的情况，而非具体跟踪某一个方法了。

2. jmap -heap pid 查看堆内存

![jmpa heap](/images/oom/jmap-heap.png)

发现老年代数据无法GC掉，占用20G+内存，使用率99%

3. jstat- gcutil pid 1000 10 查看GC情况

![jstat gc](/images/oom/jstat-gc.png)

发现每分钟一次FullGC，且无法将老年代内存清理。
此时可以确定是程序问题，怀疑某处内存泄漏，导致GC无法清除数据。


4. jmap -heap:live pid 查看堆内存中占用最多的类

![jmap histo](/images/oom/jmap-histo.png)

发现几乎都是QuizExaminne类，业务开发此时应该分析产生的代码和原因了。

5. heapdump下载堆转储文件，可以使用visualvm，MAT，JProfiler等进一步分析方法引用，此处不再展示。