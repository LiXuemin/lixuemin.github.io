---
title: Flink 1. 基本架构
date: 2020-08-30 10:32:43
categories: 流计算
tags: 
- Flink
---

### 编程模型

Flink提供了不同级别的抽象, 以开发流或批处理作业.

![layered api](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7iu5z64rj30r20bemy5.jpg)

#### ProcessFunction

* 最底层, 最具表达力的接口
* 提供了对于时间和状态的细粒度控制
* 可以用来实现许多有状态的事件驱动应用所需要的基于单个事件的复杂业务逻辑

#### DataStream API && DataSet API

* 核心API
* 提供了通用的构建模块,比如transformations, joins, aggregations, windows等
* DataSet API为有界数据集提供了额外的支持
<!--more-->
#### Table API && SQL

* 关系型API, 最简洁, 但不如核心API更具表达能力
* Table API以表为中心, 其中表在表达流数据时可以动态变化(Dynamic Table)
* table与DataStream/DataSet可以无缝切换
* 旨在简化数据分析、数据流水线和 ETL 应用的定义

### Flink体系组件

#### JobManager(Master)

协调分布式执行, 进程由以下三个组件构成:

* ResourceManager

管理 **task slots**(向资源提供平台申请, 分配空闲slot, 终止空闲的TaskManger). Flink扩展了几种不同对资源管理器: Yarn, Mesos, k8s, standalone部署.

* Dispatcher

提供了REST接口为每个提交的任务启动一个JobMaster. 

也会启动一个WebUI.

* JobMaster

管理运行时的JobGraph. Flink集群中并行运行的每个job都有自己的JobManager.

至少存在一个 JobManager, 高可用模式下, 可存在多个 JobManager, 其中一个是leader, 其它的都是standby.

#### TaskManager(Workers)

* 执行一个dataflow的task(或者特殊的subtask)、数据缓冲和datastream交换
* 运行时至少存在一个worker处理器

#### Clients

Flink提供了丰富的客户端操作提交任务和任务交互.

* Flink CommandLine
* Scala Shell, 提交Table API的任务
* SQL Client, 提交SQL任务
* RestFul API, 让用户通过http调用
* Web, 页面交互

![Flink component](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7gl0iajpj30nn0h1wga.jpg)

### 任务执行

#### Operators

出于分布式执行的目的, Flink将operator的subtask链接在一起形成task, 每个task在一个线程中执行.

好处:

* 减少线程间的的切换和基于缓存区的数据交换, 减少时延的同时提升吞吐量
下面这幅图，展示了 5 个 subtask 以 5 个并行的线程来执行:
![operators](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7hrb734lj30g00b1t9p.jpg)

#### Tasks

##### Setting Parallelism

一个特定算子的子任务(subtask)的个数被称之为其并行度(parallelism).

任务的并行度可以在Flink不同的级别设置: 

* Operator Level - source和sink均可调用`setParallelism()`方法

* Execution Enviroment Level, 调用`env.setParallelism()`方法

* Client Level, 可通过CLI中 `-p`参数,  或者代码中设置

如:

`./bin/flink run -p 10 ../examples/*WordCount-java*.jar`,

`client.run(program, 10, true);`

* System Level(set parallelism.default in flink-conf.yaml)

#### Slots and Resources

Flink 中每一个 worker(TaskManager)都是一个**JVM 进程**，它可能会在**独立的线程**上执行一个或多个 subtask。

每个 task slot 表示 TaskManager 拥有资源的**一个固定大小的子集**, slot 目前仅仅用来隔离 task 的受管理的**内存**。

通过调整 task slot 的数量，允许用户定义 subtask 之间如何互相隔离:

* 一个 TaskManager 一个 slot，意味着每个 task group 运行在独立的 JVM 中
![task-slot](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7k70qyjij30pn086q45.jpg)

* 一个 TaskManager 多个 slot  意味着更多的 subtask 可以共享同一个 JVM。而在同一个 JVM 进程中的 task 将**共享 TCP 连接(基于多路复用)和心跳消息**。它们也可能共享数据集和数据结构，因此这减少了每个 task 的负载。
![slot-sharing](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7k7s4aylj30pn0bwgoi.jpg)

> Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力，可以通过 参数 taskmanager.numberOfTaskSlots 进行配置;
> 而并行度 parallelism 是动态概念， 即 TaskManager 运行程序时实际使用的并发能力

#### Task Failure Recovery

* Restart Strategies

* Failover Strategies
