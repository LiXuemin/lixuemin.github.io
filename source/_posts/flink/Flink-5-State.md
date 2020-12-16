---
title: Flink 5. State
date: 2020-12-09 17:03:21
categories: 流计算
tags: 
- Flink
- State
---

## 前言

状态，通常是指程序运行时的中间运算结果。

通常我们在设计微服务应用的时候，为了服务的迁移和可伸缩，通常都追求无状态， 典型的设计如tomcat的session。

默认的tomcat session会存储在服务端的内存中，由客户端保存一个sessionId进行标记。而无状态的设计大致分为两类：

1. 利用客户端保存状态： 如JWT Token，token携带用户信息，每次交互时服务端验证解析token

2. 利用额外存储保存状态： 如使用redis保存session， 这样tomcat就可以实现伸缩。

那么Flink的状态有什么区别，个人理解有以下区别：

1. 首先Flink需要保存各种各样不同运算程序的中间结果,结构多种多样，并不像web应用（user,role等基本信息）那么简单，

2. Flink的状态可能每条数据处理都需要读取-计算-更新状态，web应用更多是读多写少的操作

3. 如果状态采用外部存储，那么实时性、性能上都很难保证，比如每次都要读写redis；如果状态完全采用内存管理，可靠性则无法保证

<!--more-->

所以我们可以整理一下Flink的状态保存需求：

* 基于内存，保证读写的性能，从而整体延迟不会太高

* 一套可持久化的内存备份恢复机制，不断地将内存中的状态备份；作业故障时可从备份恢复

更进一步：

* 多节点伸缩时，状态的一致性

* 状态的管理要有良好的封装，开发时不需要太关注状态的底层实现


## Flink State

### Flink StateBackend(状态后端)

Flink的每个并行任务都会把状态维护在本地。

至于状态具体的存储、访问和维护，则是由一个名为**状态后端**的可插拔组件来决定。

状态后端的主要负责两件事：

* 本地状态管理

* 将状态以检查点的形式写入远程存储

Flink内置以下三种状态后端：

MemoryStateBackend (默认)，FsStateBackend, RocksDBStateBackend

#### The MemoryStateBackend

##### 简介

将数据保存在**堆内存**中。如果配置checkpoint，会将状态快照发送到JobManager，同样在堆内存中存储。

MemoryStateBackend默认使用**异步快照**，来避免阻塞数据流。可以通过设置构造参数来关闭异步快照：
```JAVA
new MemoryStateBackend(MAX_MEM_STATE_SIZE, false);
```

##### 限制

* 每个单独状态默认限制5M内存。可以通过构造函数设置更大值。

* 状态最大不会超过`akka.framesize`设置

* aggregate状态必须适合 JobManager内存

##### 适用场景

* 本地开发和调试

* 只需要保存很小状态的任务

##### 推荐设置

官方推荐将 managed memory设为0， Flink将为程序分配最大内存。

#### The FsStateBackend

##### 简介

FsStateBackend使用文件系统存储状态。配置URL（包含类型，地址和路径）来保存：如“hdfs://namenode:40010/flink/checkpoints” or “file:///data/flink/checkpoints”.

FsStateBackend将数据保存在TaskManager的内存中。Checkpoint时，将数据快照**写入配置的文件路径中**。 元数据存在JobManager内存中（高可用模式下，存在元数据checkpoint）。

FsStateBackend默认也使用**异步快照**来进行checkpoint。同样可以关闭此特性：
```JAVA
new FsStateBackend(path, false)
```

##### 适用场景

* 状态数据很大的任务，长时间的window

* 所有要求高可用的设置

##### 推荐设置

官方推荐将 managed memory设为0， Flink将为程序分配最大内存。

#### The RocksDBStateBackend

##### 简介

RocksDBStateBackend将状态存储在TaskManager本地data目录下，自带的RocksDB数据库。当进行Checkpoint时，再将数据写入文件系统。
RocksDB无需配置，需要与FsStateBackend同样配置文件系统路径。

RocksDBStateBackend只能使用异步快照。

##### 限制

* 由于RocksDB JNI bridge API是基于字节的，单个状态的key和value不能超过2^31字节。

* 对于使用具有合并操作的状态的应用程序，例如 ListState，随着时间可能会累积到超过 2^31 字节大小，这将会导致在接下来的查询中失败

##### 适用场景

* 所有FsStateBackend适用的场景

* 目前唯一支持增量Checkpoint的方案

#### 如何选择正确的状态后端

* 生产环境尽量避免适用MemoryStateBackend，因为状态没有持久化

* FsStateBackend的性能更好，因为都是操作Java Heap上的对象； 可伸缩性受限，因为状态大小受集群可用内存限制

* RocksDBStateBackend性能稍差，因为每次操作状态都需要反序列化，并且可能从磁盘读取； 可伸缩性更好，可以对磁盘扩展，并且是唯一支持增量快照


### 算子状态

### 键值分区状态

### 有状态算子的扩缩容

## Flink Checkpoint

### 案例


示例： 利用ValueState，保存之前运算结果，并在条件满足后输出

```java
Configuration configuration = new Configuration();
StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(configuration);
env.setParallelism(2);

DataStream<Tuple2<Long, Long>> dataStream =
    env.fromElements(
        Tuple2.of(1L, 3L),
        Tuple2.of(1L, 5L),
        Tuple2.of(2L, 6L),
        Tuple2.of(2L, 2L),
        Tuple2.of(2L, 4L),
        Tuple2.of(1L, 7L));

dataStream.keyBy(0).flatMap(new CountAverageWithValueState()).print();

env.execute("Test state");
```

```java
import org.apache.flink.api.common.functions.RichFlatMapFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.util.Collector;


public class CountAverageWithValueState extends RichFlatMapFunction<Tuple2<Long, Long>, Tuple2<Long, Double>> {

    private ValueState<Tuple2<Long,Long>> countAndSum;

    @Override
    public void open(Configuration parameters) throws Exception {
        ValueStateDescriptor<Tuple2<Long,Long>> descriptor = new ValueStateDescriptor<Tuple2<Long, Long>>("average",
            Types.TUPLE(Types.LONG, Types.LONG));
        countAndSum = getRuntimeContext().getState(descriptor);
    }

    @Override
    public void flatMap(Tuple2<Long, Long> element, Collector<Tuple2<Long, Double>> out) throws Exception {
        Tuple2<Long, Long> currentState = countAndSum.value();
        if (currentState == null) {
            currentState = Tuple2.of(0L, 0L);
        }

        currentState.f0 += 1;
        currentState.f1 += element.f1;

        countAndSum.update(currentState);

        if (currentState.f0 >= 3) {
            double avg = (double)currentState.f1 / currentState.f0;
            out.collect(Tuple2.of(element.f0, avg));
            countAndSum.clear();
        }
    }
}
```