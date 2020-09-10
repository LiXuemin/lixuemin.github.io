---
title: 数据收集(1)-Flume
date: 2020-05-08 16:20:58
categories: 读书笔记
tags:
- 笔记
- Flume
---
## 一. 背景

日志收集面临诸多难题：

- 数据源种类繁多： 各服务产生日志格式不同，产生的方式也不同（本地文件，HTTP远程发送等）
- 数据源是物理分布的： 日志是在分布式机器上，甚至跨机房
- 流式的、不间断产生： 有些日志需要实时或近实时收集到，以供后端分析挖掘
- 对可靠性有一定要求
<!--more-->
## 二. Flume简介

### Flume OG

- Flume是由Cloudera公司开源的，一种分布式的高可靠的中间件。
- 能够对不同数据源的海量日志进行高效收集、聚合、移动，最后存储到后端存储系统中。

### Flume NG

- `Flume OG` (Original Generation)， 对应Flume 0.9.x及之前版本，已弃用， 看到相关帖子可以跳过了。
- `Flume NG` (Next/New Generation)，对应Flume 1.x系列版本，目前应用广泛。

去掉了中心化组件master及协调组件Zookeeper。

## 三. 基础概念

Flume的数据流是通过一系列称为**Agent**的组件构成的。

![基础概念](/images/flume/flume01.png)

一个Agent可从客户端（或者前一个Agent），经过拦截器（可选）、路由等操作后，传递给下一个或多个Agent，直到抵达指定的目标系统。

### Event

Flume将数据流水线中传递的数据称为 Event.

Event是Flume的基本数据单位.

每个Event由以下两部分组成：

- 头部 —— 一系列key:value对，可用于数据路由
- 字节数组 ——封装了实际要传递的数据内容， 通常使用Avro,Thrift,Protobuf等对象序列化而成

Event可由专门的Client生成，Client将要发送的数据封装成Event对象，并调用Flume提供的SDK发送给Agent.

### Agent组件构成

Agent内存主要由三个组件构成，分别是Source, Channel 和 Sink.

> Sink做动词时翻译为下沉， 做名词时翻译为 **水槽，水池，沟渠**.
一目了然，数据像水流(封装为Event)，有源头(Source)，有管道(Channel)，最终通过沟渠(Sink)汇入湖海(HDFS，Hbase等存储)。
当然Agent的Sink(沟渠)也可以将Event（水流）发送到另一个Agent的Source(水源)。

- Source

Flume数据流中接收Event的组件。

通常从Client程序或者上一个Agent的Sink接收数据，并写入一个或多个Channel。

Flume提供了很多Source实现：

来自网络:

- `Avro/Thrift Source`，接收不同RPC客户端发送的数据
- `Kafka Source`
- `Syslog Source, TCP/UDP`
- `HTTP Source`

    ...

来自文件

- `Exec Source`，执行指定Shell，并从标准输出中获取数据。如“tail -f xxx.log”。容错性很差
- `Spooling Directory Source` ,监控目录池下文件变化，并将新文件近实时传入Channel。但是文件拷贝到目录下不能修改，目录下不能包含子目录。
- `Taildir Source` , 实时监控，实时读取新增数据，断点续传

    ...

- Channel

缓存区，暂存Source写入的Event，直到被 Sink 发送出去。

平衡Source和Sink之间的速度差异：

如果Sink处理吞吐量 < Source发送吞吐量

目前Flume主要有以下几种Channel实现：

- `Memory Channel`，内存中缓存Event，性能高，但断电或者内存不足时有问题
- `File Channel`，磁盘文件中缓存，性能有所下降(建议File Channel目录跟日志目录在不同磁盘上, 提高效率)
- `JDBC Channel`，适用于对故障恢复要求极高的场景
- `Kafka Channel`

    ...

- Sink

接收Channel数据，并发送给下一个Agent的Source，或者发送给存储系统。

- `HDFS Sink`
- `HBase Sink`，支持同步，异步两种写入方式
- `Hive Sink`
- `ElasticSearchSink/MorphlineSolrSink`，写入ES，Solr
- `Kafaka Sink`
- `Logger Sink`
- `Http Sink`

  ...

## 四、高级组件

为了方便用户更灵活控制数据流，Flume还允许用户设置其它组件：

- Interceptor

![interceptor](/images/flume/flume02.png)

Interceptor组件作用在Source和Channel之间, 允许修改或丢弃传输过程中的Event.

校验Event数据格式, 给Event增加header(默认只有message).

需要实现 `org.apache.flume.interceptor.Interceptor`接口。

可配置多个Interceptor，配置声明的顺序就是执行顺序。

"preserveExisting"为true时, 若header已存在, 则不替换

- `Timestamp Interceptor`，"timestamp": 当前时间
- `Host Interceptor`, "host": agent所在IP;
- `Static Interceptor`，增加固定的键值对
- `Remove Header Interceptor`
- `UUID Interceptor`
- `Regx Filtering Interceptor`

    ...

- Channel Selector

Channel Selector允许Source选择一个或多个符合条件的Channel，并将Event写入Channel.

- `Replicating Channel Selector`(default)， 将相同数据导入多个Channel中
- `Multiplexing Channel Selector`，根据Event header值动态路由
- `Custom Channel Selector`，用户自定义

- Sink Processor

`Sink Group`：Flume允许将多个Sink组装在一起形成一个逻辑实体。

Sink Processor在 Sink Group基础上提供负载均衡以及容错的功能。

- `Default Sink Processor`
- `Failover Sink Processor`
- `Load Balancing Sink Processor`，目前支持轮询和随机
- `Custom Sink Processor`

## 五、Flume的特性

### 可靠性保证

当节点出现故障时，日志能够被传送到其他节点上而不会丢失。Flume提供了三种级别的可靠性保障，从强到弱依次分别为：

- end-to-end：收到数据agent首先将event写到磁盘上，当数据传送成功后，再删除；如果数据发送失败，可以重新发送。
- Store on failure：这也是scribe采用的策略，当数据接收方crash时，将数据写到本地，待恢复后，继续发送
- Besteffort：数据发送到接收方后，不会进行确认

### 事务

Flume 使用两个独立的事务:

- Put事务, 负责Source → Channel的Event传递
    1. doPut: 数据写入临时缓冲区putList
    2. doCommit: 检查Channel内存队列是否足够合并
    3. doRollback: channel内存队列空间不足, 回滚数据
- Take事务, 负责 Channel → Sink的Event传递
    1. doTake : 数据读取到临时缓冲区takeList
    2. doCommit: 若数据全部发送成功, 则清除takeList
    3. doRollback: 发送过程中出现任何异常, 将takeList中的数据归还给 Channel内存队列


## 六、Flume应用场景

1. 从集群中每个节点都读取日志, 最终汇聚到一个agent并存储

![读取集群日志汇总](/images/flume/flume03.png)

2. Multiplexing(多路) Agent

![多路](/images/flume/flume04.png)

一般有两种使用方式:

- 复制(Replication), 数据复制成相同的多份发送到每个Channel
- 分流(Multiplexing), Selector根据header的值确定传递到哪一个Channel
