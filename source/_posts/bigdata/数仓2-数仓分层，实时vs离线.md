---
layout: bigdata
title: 数仓2. 数仓分层，实时vs离线
date: 2020-10-14 21:47:20
tags:
- 数据仓库
- 数仓分层
---

## 数据仓库概念

一听到数据仓库的概念, 我们第一反应会问到数据库与数据仓库的区别在哪里?

我的理解是这样的:

## 数据仓库分层
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjw482colsj30mj0bf75s.jpg)
整体来讲, 可以分为数据输入, 数据分析, 数据输出

ODS, 备份一份原始数据

DWD, 负责数据ETL

DWS, 聚合层 按天

DWT, 聚合层 累积

ADS, 指标

## 离线数仓 VS 实时数仓

![](https://i.loli.net/2020/11/27/irHSFayM4wXhzRN.png)

### Lambda, Kappa

## 数仓技术选型


### 输入层

### OLAP数据引擎