---
title: Flink-2. 流计算API基础
date: 2020-09-05 16:15:49
categories: 流计算
tags: 
- Flink
---

## Flink运行模型

![runtime](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7lszqn0xj30jg04i74i.jpg)
以上为 Flink 的运行模型，Flink 的程序主要由三部分构成，分别为 Source、 Transformation、Sink。DataSource 主要负责数据的读取，Transformation 主要负责对 属于的转换操作，Sink 负责最终数据的输出。

每个 Flink 程序都包含以下的若干流程:

1. 获得一个执行环境;(Execution Environment)
2. 加载/创建初始数据;(Source)
3. 指定转换这些数据;(Transformation)
4. 指定放置计算结果的位置;(Sink)
5. 触发程序执行

如下图所示:
<!--more-->
![stream dataflow](https://tva1.sinaimg.cn/large/007S8ZIlly1gj7ls5f2oej30hl0dswfz.jpg)

## Enviroment

**执行环境 StreamExecutionEnvironment 是所有 Flink 程序的基础。**

创建一个执行环境，表示当前执行程序的上下文。

* 如果程序独立调用, 如IDE中Run, 返回本地执行环境
* 如果提交到集群, 则返回此集群的执行环境

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

## Source

数据源.

* 内置的数据源: 基于File, Socket, Collection等.

* addSource, 自定义数据源, 官方提供了常用的kafka, rabbitmq等, 也可以自己开发

eg: `env.addSource(new FlinkKafkaConsumer010<>(...))`

## Sink

Data Sinks消费DataStreams中的数据, 并且输出到file, socket, 外部系统中.

* 内置可以输出到txt, csv, socket, 直接打印等
* addSink - 扩展sink方法, 通过connectors消费到外部系统

## Transformations

转换算子.

Basic Transformations

* Map, 一对一转换
* Filter, 过滤
* FlatMap, 一对零/一/多转换

KeyedStream Transformations

* KeyBy, 基于key对流(内部使用hash函数)进行分区
* Aggregations, 聚合操作,如min, max, sum等
* Reduce, 返回单个结果值

Multistream Transformation

* Union, 组合流
* Connect, coMap, coFlatMap
* Split & select, split拆分流, select从拆分流中选择特定流

Distribution Transformation

* Random
* Round-Robin
* Rescale
* Broadcast
* Global
* Custom
