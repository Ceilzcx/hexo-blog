---
title: Flink
date: 2021-10-09 15:10:11
tags: 大数据
---


## what is Flink？

> **分布式** 处理引擎

### 流

> 无边界 / 有边界 的 **有状态** 的计算

+ **无边界**：只定义开始，没有结束，数据摄入后立即执行（输入无限）
+ **有边界**：定义开始和结束。可以数据摄入后立即立即执行（**实时**），也可以等待全部输入（存储在存储引擎中）后批量执行（**历史记录**）

任务 —— 并行处理

### 状态

> 只有在每一个单独的事件上进行转换操作的应用才不需要状态

+ **多种状态基础类型**：数据类型（value、map、list等）
+ **`State Backend`**：管理状态。内存 / `RocksDB`
+ **精确一次语义**：处理故障，保证状态一致性
+ **超大数据量状态**：利用其异步以及增量式的 checkpoint 算法，存储数 TB 级别的应用状态。
+ **可弹性伸缩的应用**：在更多或更少的工作节点上对状态进行重新分布，支持有状态应用的分布式的横向伸缩。

### 时间

> 事件总是在特定时间点发生，所以大多数的事件流都拥有事件本身所固有的时间语义

+ **事件时间模式**：本身自带的时间戳进行结果的计算。保证准确性和一致性

  为什么自带时间戳？例如窗口模式，将同一个范围的时间戳放在一个bucket里面

+ **Watermark 支持**：衡量事件时间进展。平衡处理延时和完整性的灵活机制（Future）

  什么是watermark？简单的举例：时间戳为1-10的数据按顺序进入task Manager执行，如果按照5的范围设置，那么等到5的时间戳到达说明1-5的数据都已经拿到，关闭对应的bucket，执行任务；但是数据存在乱序的可能，可能5的数据已经拿到，但是3的数据在后面，如果关闭了bucket那么3的数据就丢失，因此可以通过设置watermark，如果设置watermark为2，拿到5的数据时，判断5-2=3，不关闭bucket，等到拿到7的数据关闭1-5的bucket。因此设置合理的watermark可以解决大部分低延迟的数据。

+ **迟到数据处理**：当以带有 watermark 的事件时间模式处理数据流时，在计算完成之后仍会有相关数据到达。这样的事件被称为迟到事件。Flink 提供了多种处理迟到数据的选项，例如将这些数据重定向到旁路输出（side output）或者更新之前完成计算的结果。

+ **处理时间模式**：处理时间语义。处理时间模式根据处理引擎的机器时钟触发计算，一般适用于有着严格的低延迟需求，并且能够容忍近似结果的流处理应用。



### 分层 API

+ High-level Analytics API：只需要写 SQL / Table API
+ DataStream API：写数据流和批处理，可以调用streams和windows
+ ProcessFunction：Stateful Event-Driven Applications，可以调用events、state和time



### 运行架构

#### 作业管理器（JobManager）





### 流处理

#### 特点

+ event：事件触发，具有极强的时间性（事件发生、事件进入、事件处理时间 等）
+ Stream：事件流，无界
+ Process：流处理，





### 双流Join操作

+ join()
+ coGroup()
+ intervalJoin()

#### 1、join()

> 对应 mysql 的 inner join

通过一个窗口，进行join操作，简单易用。

存在问题：一个流的数据存在延迟时，另一个流的数据没有对应的join数据。

#### 2、coGroup()

> 对应 mysql 的 left/right outer join

双重循环

#### 3、intervalJoin()

按照指定字段以及右流相对左流偏移的时间区间进行关联，即：

> right.timestamp ∈ [left.timestamp + lowerBound; left.timestamp + upperBound]









## 架构和源码

### flink-connector-jdbc

#### upsert

作为sink向外部数据库写入数据时，如果使用DDL定义的主键，连接器将在upsert模式下操作（需要保证幂等性），否则使用append操作（这时插入主键相同的数据会出现主键冲突的异常）。

#### cache

JDBC 可以在临时连接中用作查找源。

默认情况下，不确定缓存查找。可以设置 `lookup.cache.max-rows` 和 `lookup.cache.ttl` 设置启动它.

使用缓存存在数据不是最新的问题，因此需要合理设置最大行和过期时间。



#### `catalog`

> 目录

将 database → table 的形式转出类似目录的形式



#### `dialect`

> 方言，不同 JDBC 语法的差异

upsert操作参考：`JdbcDialect.getUpsertStatement`



#### table

Source、Sink、Function

TableSchema 在 table-common包中





#### package(version：1.13)

maven包可能不存在，在 `settings.xml` 添加国际镜像

```tex
<mirror>
	<id>mapr-public</id>
	<mirrorOf>mapr-releases</mirrorOf>
	<name>mapr-releases</name>
	<url>https://maven.aliyun.com/repository/mapr-public</url>
</mirror>
```

打包执行 `mvn clean install -DskipTests -Dfast -T 4 -Drat.skip=true ` ，最后一句一定要加，用来跳过license，不然会报下面错误

```tex
Failed to execute goal org.apache.rat:apache-rat-plugin:0.12:check (default) on project flink-parent: Too many files with unapproved license: 4 See RAT report in: D:\ffffff\flink-release-1.10.0\flink-release-1.10.0\target\rat.txt
```

flink-runtime-web下载node速度较慢，下载超时失败。修改 pom.xml 的配置信息，如果已经操作，需要删除 web-dashboard 的 node modules

```xml
// 修改
<arguments>ci --cache-max=0 --no-save</arguments>
// 替换
<arguments>install -registry=https://registry.npm.taobao.org --cache-max=0 --no-save</arguments>
```

代码规范

```shell
mvn spotless:apply
```

node 添加 其他下载源

```xml
<configuration>
	<nodeDownloadRoot>https://registry.npm.taobao.org/dist/</nodeDownloadRoot>
	<npmDownloadRoot>https://registry.npmjs.org/npm/-/</npmDownloadRoot>
	<nodeVersion>v10.9.0</nodeVersion>
</configuration>
```

局部打包，例如我要打包 flink-connector 模块，`-pl`：指定需要打包的模块，`-am`：加载依赖模块

```shell
mvn clean install -pl flink-connectors -am -Drat.skip=true
```

