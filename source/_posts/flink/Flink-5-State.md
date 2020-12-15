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

将数据保存在**堆内存**中。

如果配置checkpoint，会将状态快照发送到JobManager，同样在堆内存中存储。


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