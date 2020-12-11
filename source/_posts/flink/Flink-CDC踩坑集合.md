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