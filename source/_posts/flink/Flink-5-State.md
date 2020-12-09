---
title: Flink 5. State
date: 2020-12-09 17:03:21
tags:
---

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