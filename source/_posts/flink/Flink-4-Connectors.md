---
title: Flink 4. Connectors
date: 2020-09-17 17:57:04
categories: 流计算
tags: 
- Flink
---

### DataStreeam Connector

#### 预定义的Source

* File-based:

```java
readTextFile(path)
readFile(fileInputFormat, path)
readFile(fileInputFormat, path, watchType, interval, pathFilter, typeInfo)
```

* Socket-based:

```java
socketTextStream()
```

* Collection-based:

```java
fromCollection(Collection)
fromCollection(Iterator, Class)
fromElements(T ...)
fromParallelCollection(SplittableIterator, Class)
generateSequence(from, to) 
```

* Custom:

```java
addSource()
```

#### 预定义的Sink

```java
writeAsText() / TextOutputFormat

writeAsCsv(...) / CsvOutputFormat

print() / printToErr()

writeUsingOutputFormat() / FileOutputFormat

writeToSocket()

addSink()
```

#### 附带连接器

* Apache Kafka (source/sink)
* Apache Cassandra (sink)
* Amazon Kinesis Streams (source/sink)
* Elasticsearch (sink)
* FileSystem（包括 Hadoop ） - 仅支持流 (sink)
* FileSystem（包括 Hadoop ） - 流批统一 (sink)
* RabbitMQ (source/sink)
* Apache NiFi (source/sink)
* Twitter Streaming API (source)
* Google PubSub (source/sink)
* JDBC (sink)

#### Apache Bahir 中的连接器
Flink 还有些一些额外的连接器通过 Apache Bahir 发布, 包括:

* Apache ActiveMQ (source/sink)
* Apache Flume (sink)
* Redis (sink)
* Akka (sink)
* Netty (source)

#### 连接Flink的其它方法

##### 异步 I/O

使用connector并不是唯一可以使数据进入或者流出Flink的方式。 一种常见的模式是从外部数据库或者 Web 服务查询数据得到初始数据流，然后通过 Map 或者 FlatMap 对初始数据流进行丰富和增强。 Flink 提供了异步 I/O API 来让这个过程更加简单、高效和稳定。

##### 可查询状态（Queryable State Beta）
当 Flink 应用程序需要向外部存储推送大量数据时会导致 I/O 瓶颈问题出现。在这种场景下，如果对数据的读操作远少于写操作，那么让外部应用从 Flink 拉取所需的数据会是一种更好的方式。 可查询状态 接口可以实现这个功能，该接口允许被 Flink 托管的状态可以被按需查询。

### Table Connnector

#### mysql-cdc

业务Mysql -> Flink CDC -> 数仓贴源层

cdc采集端配置

```SQL
CREATE TABLE `ods_table_name` (
  `PK_ID` BIGINT NOT NULL,
  `TITLE` STRING,
  ...
) WITH (
 'connector' = 'mysql-cdc',
 'hostname' = 'localhost',
 'port' = '3306',
 'username' = 'root',
 'password' = 'root',
 'database-name' = 'dbname',
 'table-name' = 'mysql_table_name'
);
```

CDC输出端建表

```SQL
CREATE TABLE `dwd_resource_label` (
  `PK_ID` BIGINT PRIMARY KEY NOT ENFORCED,
  `TITLE` STRING,
  ...
) WITH (
 'connector' = 'jdbc',
 'url' = 'jdbc:mysql://localhost:3306/dbname?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8',
 'table-name' = 'mysql_table_name_destination',
 'username' = 'root',
 'password' = 'root'
);
```

同步

```SQL
INSERT INTO dwd_resource_label SELECT * FROM ods_resource_label;
```

### Dataset Connector

