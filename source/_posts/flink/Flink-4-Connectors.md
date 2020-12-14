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

Table API和SQL程序，都支持连接并读写外部系统。 对于Table连接器，除了要了解支持的**连接器**， 还应该了解其支持的**表格式**。

#### 连接器

以Flink1.12为例，下面是其Table API & SQL支持的连接器类型。

|Name	|Version|	Source|	Sink|
|---- |----|----|----|
|Filesystem		    |                   |Bounded and Unbounded Scan,          |Lookup	Streaming Sink, Batch Sink|
|Elasticsearch	  |6.x & 7.x	        |Not supported	                      |Streaming Sink, Batch Sink|
|Apache Kafka	    |0.10+	            |Unbounded Scan	                      |Streaming Sink, Batch Sink|
|Amazon Kinesis   |           		    |Unbounded Scan	                      |Streaming Sink|
|JDBC		          |                   |Bounded Scan, Lookup	                |Streaming Sink, Batch Sink|
|Apache HBase	    |1.4.x & 2.2.x    	|Bounded Scan, Lookup	                |Streaming Sink, Batch Sink|
|Apache Hive	    |                 	|Unbounded Scan, Bounded Scan, Lookup	|Streaming Sink, Batch Sink|


#### 格式

表格式是一种存储格式，定义了如何把二进制数据映射到表的列上。

|格式	          | 支持的连接器
|----           |----
|CSV	          |Apache Kafka, Upsert Kafka, Amazon Kinesis Data Streams, Filesystem
|JSON	          |Apache Kafka, Upsert Kafka, Amazon Kinesis Data Streams, Filesystem, Elasticsearch
|Apache Avro	  |Apache Kafka, Upsert Kafka, Amazon Kinesis Data Streams, Filesystem
|Confluent Avro	|Apache Kafka, Upsert Kafka
|Debezium CDC	  |Apache Kafka, Filesystem
|Canal CDC	    |Apache Kafka, Filesystem
|Maxwell CDC	  |Apache Kafka, Filesystem
|Apache Parquet	|Filesystem
|Apache ORC	    |Filesystem
|Raw	          |Apache Kafka, Upsert Kafka, Amazon Kinesis Data Streams, Filesystem

#### 使用示例

```SQL
CREATE TABLE MyUserTable (
  -- declare the schema of the table
  `user` BIGINT,
  `message` STRING,
  `rowtime` TIMESTAMP(3) METADATA FROM 'timestamp',    -- use a metadata column to access Kafka's record timestamp
  `proctime AS PROCTIME(),    -- use a computed column to define a proctime attribute
  WATERMARK FOR `rowtime` AS `rowtime` - INTERVAL '5' SECOND    -- use a WATERMARK statement to define a rowtime attribute
) WITH (
  -- declare the external system to connect to
  'connector' = 'kafka',
  'topic' = 'topic_name',
  'scan.startup.mode' = 'earliest-offset',
  'properties.bootstrap.servers' = 'localhost:9092',
  'format' = 'json'   -- declare a format for this system
)
```

### Dataset Connector

#### 文件系统

#### Avro支持

#### MongoDB
