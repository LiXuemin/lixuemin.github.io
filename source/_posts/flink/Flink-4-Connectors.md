---
title: Flink 4. Connectors
date: 2020-09-17 17:57:04
categories: 流计算
tags: 
- Flink
---

### DataStreeam Connector

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

