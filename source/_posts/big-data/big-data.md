---
title: big-data
date: 2021-10-29 14:23:31
tags: 大数据
---

## 大数据知识



### OLAP 和 OLTP 的概念和比较

+ OLAP：联机分析处理。数据仓库的主要应用，支持复杂的数据操作。
+ OLTP：连接事务处理。传统的关系型数据库的主要应用，主要是基本的事务处理。强调数据的实时性和内存效率，以及并发的操作

|          | OLTP       | OLAP                           |
| -------- | ---------- | ------------------------------ |
| 数据     | 最新的数据 | 历史数据；聚合、多维集成的数据 |
| 工作单位 | 事务       | 查询                           |
| 时间     | 实时性     | 存在一定延迟                   |
| 应用     | 数据库     | 数据仓库                       |



### `Hadoop`、`HDFS`、`Hive`、`HBase`的关系

+ `Hadoop`：分布式计算的开源框架

+ `HDFS`：分布式文件系统（`Hadoop`三大组件之一）

+ `Hive`：存储处理数据的 `sql`，`Hive`会将 `sql` 转化为 `MapReduce` 程序。

  本身并不存储数据，完全依赖于 `HDFS`  和 `MapReduce`。

+ `HBase`：基于 `HDFS` 的数据库，`NoSQL` 数据库，用于海量数据数据（亿）的随机查询

  物理表，提供一个超大的内存 `Hash` 表，通过他存储索引，方便查询操作

+ `Sqoop`：为 `HBase` 提供了方便的 `RDBMS` 数据导入功能，使得传统的数据向 `HBase` 迁移变得非常方便

![hadoop.png]()
