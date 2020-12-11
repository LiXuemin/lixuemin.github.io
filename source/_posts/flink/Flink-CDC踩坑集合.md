---
title: Flink CDC踩坑集合
date: 2020-12-11 17:02:47
categories: 流计算
tags: 
- Flink
- 踩坑
---

1. 使用flink sql 时，提前引入flink-json依赖

异常信息

```
Caused by: org.apache.flink.table.api.ValidationException: Could not find any factories that implement 'org.apache.flink.table.factories.DeserializationFormatFactory' in the classpath.
	at org.apache.flink.table.factories.FactoryUtil.discoverFactory(FactoryUtil.java:229)
	at org.apache.flink.table.factories.FactoryUtil$TableFactoryHelper.discoverOptionalFormatFactory(FactoryUtil.java:538)
	at org.apache.flink.table.factories.FactoryUtil$TableFactoryHelper.discoverOptionalDecodingFormat(FactoryUtil.java:426)
	at org.apache.flink.table.factories.FactoryUtil$TableFactoryHelper.discoverDecodingFormat(FactoryUtil.java:413)
	at org.apache.flink.streaming.connectors.kafka.table.KafkaDynamicTableFactoryBase.createDynamicTableSource(KafkaDynamicTableFactoryBase.java:73)
	at org.apache.flink.table.factories.FactoryUtil.createTableSource(FactoryUtil.java:122)
	  
```

解决方案： pom文件中引入

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-json</artifactId>
    <version>1.11.0</version>
</dependency>
```

2. Flink 1.11版本后简化了 connector 参数

Flink由于发展太快（2020年已经发布了1.10，1.11，1.12三个大版本），很多2020年初的blog提供教程已经面临失效

以 Kafka 为例，在 1.11 版本之前用户的 DDL 需要声明成如下方式

```sql
 CREATE TABLE user_behavior (
  ...
) WITH (
  'connector.type'='kafka',
  'connector.version'='universal',
  'connector.topic'='user_behavior',
  'connector.startup-mode'='earliest-offset',
  'connector.properties.zookeeper.connect'='localhost:2181',
  'connector.properties.bootstrap.servers'='localhost:9092',
  'format.type'='json'
);
```

而在 Flink SQL 1.11 中则简化为

```sql
CREATE TABLE user_behavior (
  ...
) WITH (
  'connector'='kafka',
  'topic'='user_behavior',
  'scan.startup.mode'='earliest-offset',
  'properties.zookeeper.connect'='localhost:2181',
  'properties.bootstrap.servers'='localhost:9092',
  'format'='json'
);
```

详细变更见[FLIP-122](https://cwiki.apache.org/confluence/display/FLINK/FLIP-122%3A+New+Connector+Property+Keys+for+New+Factory)

3. flink-sql-connector-jdbc声明表时，必须指定主键

否则会报异常：

```
java.lang.IllegalStateException: please declare primary key for sink table when query contains update/delete record.
	at org.apache.flink.util.Preconditions.checkState(Preconditions.java:195)
	at org.apache.flink.connector.jdbc.table.JdbcDynamicTableSink.validatePrimaryKey(JdbcDynamicTableSink.java:72)
	at org.apache.flink.connector.jdbc.table.JdbcDynamicTableSink.getChangelogMode(JdbcDynamicTableSink.java:63)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkChangelogModeInferenceProgram$SatisfyModifyKindSetTraitVisitor.visit(FlinkChangelogModeInferenceProgram.scala:120)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkChangelogModeInferenceProgram.optimize(FlinkChangelogModeInferenceProgram.scala:50)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkChangelogModeInferenceProgram.optimize(FlinkChangelogModeInferenceProgram.scala:39)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkGroupProgram$$anonfun$optimize$1$$anonfun$apply$1.apply(FlinkGroupProgram.scala:63)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkGroupProgram$$anonfun$optimize$1$$anonfun$apply$1.apply(FlinkGroupProgram.scala:60)
	at scala.collection.TraversableOnce$$anonfun$foldLeft$1.apply(TraversableOnce.scala:157)
	at scala.collection.TraversableOnce$$anonfun$foldLeft$1.apply(TraversableOnce.scala:157)
	at scala.collection.Iterator$class.foreach(Iterator.scala:891)
	at scala.collection.AbstractIterator.foreach(Iterator.scala:1334)
	at scala.collection.IterableLike$class.foreach(IterableLike.scala:72)
	at scala.collection.AbstractIterable.foreach(Iterable.scala:54)
	at scala.collection.TraversableOnce$class.foldLeft(TraversableOnce.scala:157)
	at scala.collection.AbstractTraversable.foldLeft(Traversable.scala:104)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkGroupProgram$$anonfun$optimize$1.apply(FlinkGroupProgram.scala:60)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkGroupProgram$$anonfun$optimize$1.apply(FlinkGroupProgram.scala:55)
	at scala.collection.TraversableOnce$$anonfun$foldLeft$1.apply(TraversableOnce.scala:157)
	at scala.collection.TraversableOnce$$anonfun$foldLeft$1.apply(TraversableOnce.scala:157)
	at scala.collection.immutable.Range.foreach(Range.scala:160)
	at scala.collection.TraversableOnce$class.foldLeft(TraversableOnce.scala:157)
	at scala.collection.AbstractTraversable.foldLeft(Traversable.scala:104)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkGroupProgram.optimize(FlinkGroupProgram.scala:55)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkChainedProgram$$anonfun$optimize$1.apply(FlinkChainedProgram.scala:62)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkChainedProgram$$anonfun$optimize$1.apply(FlinkChainedProgram.scala:58)
	at scala.collection.TraversableOnce$$anonfun$foldLeft$1.apply(TraversableOnce.scala:157)
	at scala.collection.TraversableOnce$$anonfun$foldLeft$1.apply(TraversableOnce.scala:157)
	at scala.collection.Iterator$class.foreach(Iterator.scala:891)
	at scala.collection.AbstractIterator.foreach(Iterator.scala:1334)
	at scala.collection.IterableLike$class.foreach(IterableLike.scala:72)
	at scala.collection.AbstractIterable.foreach(Iterable.scala:54)
	at scala.collection.TraversableOnce$class.foldLeft(TraversableOnce.scala:157)
	at scala.collection.AbstractTraversable.foldLeft(Traversable.scala:104)
	at org.apache.flink.table.planner.plan.optimize.program.FlinkChainedProgram.optimize(FlinkChainedProgram.scala:57)
	at org.apache.flink.table.planner.plan.optimize.StreamCommonSubGraphBasedOptimizer.optimizeTree(StreamCommonSubGraphBasedOptimizer.scala:164)
	at org.apache.flink.table.planner.plan.optimize.StreamCommonSubGraphBasedOptimizer.doOptimize(StreamCommonSubGraphBasedOptimizer.scala:80)
	at org.apache.flink.table.planner.plan.optimize.CommonSubGraphBasedOptimizer.optimize(CommonSubGraphBasedOptimizer.scala:77)
	at org.apache.flink.table.planner.delegation.PlannerBase.optimize(PlannerBase.scala:279)
	at org.apache.flink.table.planner.delegation.PlannerBase.translate(PlannerBase.scala:164)
	at org.apache.flink.table.api.internal.TableEnvironmentImpl.translate(TableEnvironmentImpl.java:1264)
	at org.apache.flink.table.api.internal.TableEnvironmentImpl.executeInternal(TableEnvironmentImpl.java:700)
	at org.apache.flink.table.api.internal.TableEnvironmentImpl.executeOperation(TableEnvironmentImpl.java:787)
	at org.apache.flink.table.api.internal.TableEnvironmentImpl.executeSql(TableEnvironmentImpl.java:690)
    ···
```

4. 使用flink-mysql-cdc时，请注意检查线上数据库binlog-format属性，另外要给用户授权

```sql
SET GLOBAL binlog_format = 'ROW';
```

```sql
GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'user' IDENTIFIED BY 'password';
```

5. 接收到MIXED或STATEMENT格式日志退出

6. 目标表要注意清除外键依赖