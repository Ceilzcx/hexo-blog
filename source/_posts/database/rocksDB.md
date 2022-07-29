---
title: rocksDB
date: 2022-05-30 10:40:40
tags: 数据库
---

## RocksDB

适用环境：内存、**Flash**、hard disks、HDFS



### 数据结构

+ memtable（**内存**数据结构）



+ sstfile（磁盘）

有序存储，方便查询

+ logfile（顺序写）



数据先写入 `memtable`，部分请求内容写入 `logfile`（WAL：Write-Ahead Logging）

`memtable` 内存满之后，执行 flush 操作，将数据转移到 `sstfile`，同时删除 logfile 的数据



RocksDB 的 key 和 value 完全是 byte stream（无长度限制）

**DB的所有数据按照key有序存储**（因此 `iterator` 可以支持rangeScan查询），`compare` 方法可以用户自定义

`Snapshot` 可以创建一个时间点的快照（`Get` 和 `Iterator` 可以执行在某一个 `Snapshot` 上）



#### 读放大 和 写放大

+ 数据存入内存，构建成一棵树

+ 内存数据越来越大，flush 到磁盘

+ 磁盘定期执行 merge 操作，合并成为一棵更大的树（L0 → LN，L0的热度最高，LN的热度最低）

读取数据需要一步步从内存到硬盘，因此称为读放大。



#### 同步写 和 异步写

+ 同步写：写入磁盘返回
+ 异步写：写入内存返回，写入磁盘是异步操作

异步写操作系统崩溃会丢掉部分数据（一般不会发生），同步写效率比异步写差千倍左右。



#### 事务（Transations）

支持乐观和悲观锁



#### Column Family

> 列族

每个key都与唯一的列族结合

跨列族的原子性操作

快速删除/添加列族（**共享WAL日志，不共享`memtable`和`table`文件**）

一个列族 flush 后，创建新的 WAL 日志，老的 WAL 不删除，等到所有的列族都 flush 后才会删除老的 WAL 文件

