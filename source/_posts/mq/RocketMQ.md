---
title: RocketMQ
date: 2021-10-14 16:38:36
tags: 消息队列
---



## 设计架构

### Producer

> 消息提供/发布者，支持分布式和集群部署

通过 `NameServer`，获取 `Broker` 路由信息，通过负载均衡模块选择对应的 `Broker` 集群队列，并发布消息。

投递的过程支持快速失败并且低延迟。

### Consumer

> 消息消费者，支持分布式和集群部署

支持 push / pull 进行消费；支持集群（默认消费模式）和广播形式消费

+ 同一个 `ConsumerGroup` 下，一条消息消费一次
+ 同一个 `ConsumerGroup` 下，一条消息每个实例都消费一次

实时消息订阅（订阅topic）

### NameServer

> topic 路由注册中心，（类似dubbo的zookeeper），支持broker的注册和发现



**AP的设计：Availability（可用性）+ Partition Tolerance（分区容错性）**

`NameServer` **节点之间并不通信**，各个时刻数据没法保持一致（不遵循 Consistency（一致性））

   

**Broker管理**：接受Broker集群注册信息并保存作为路由信息的基本数据

**提供健康检查**：检查Broker是否存活

   

*每个 `NameServer` 将保存关于Broker集群的整个路由信息和用于客户端查询的队列信息，然后 `Producer` 和 `Conumser` 通过 `NameServer` 就可以知道整个Broker集群的路由信息，从而进行消息的投递和消费。*

如何注册topic？

+ 手动创建，指定对应的broker和消息队列数量进行创建

+ 自动创建

  ```shell
  autoCreateTopicEnable = true
  ```

  + 生产发送消息时，如果指定的 Topic 不存在，`NameServer` 会返回一个【默认主题：`TBW102`】的路由信息，使得生产者能够正常发生消息

  + Broker 收到消息后，发现消息对应 Topic 不存在，且 Broker 允许自动创建 Topic，则会为消息创建 Topic ，并定时把路由信息同步至 `NameServer`

  + 生产者也会定时从 `NameServer` 同步最新的路由信息，缓存至本地

  + 后续生产者发送消息时，就可以从本地的缓存中查询到对应 Topic 的路由信息了



### BorkerServer

> broker主要负责消息的存储、传递和查询以及服务高可用保证

broker 分为 Master 和 Slave，对应关系通过相同 `BrokerName`，不同的 `BrokerId` 来定义，`BrokerId = 0` 表示Master；broker支持一主多从，但是**只有 `brokerId = 1` 的从服务器才会参与消息的读负载** 。

   

broker定时将topic信息注册到 `NameServer`



### Topic和Tag

+ topic 为 消费 和 提供者 订阅的一种主题
+ tag 为 topic 更细粒度的一种划分。例：topic = oder，你想要获取不同类型的订单，可以定义不同的tag



## 网络部署

### 特点 

+ Producer与 `NameServer` 集群的一个节点建立连接（随机），获取topic路由信息，与**Master**建立长连接，并定时发送心跳
+ Consumer与 `NameServer` 集群的一个节点建立连接（随机），获取topic路由信息，与**Master和Slave**建立长连接，并定时发送心跳

### 启动流程

+ NameServer 启动，等待Broker、Producer、Consumer连接
+ Broker启动，跟所有的NameServer保持长连接，定时发送心跳包。心跳包中包含当前Broker信息(IP+端口等)以及存储所有Topic信息。注册成功后，NameServer集群中就有Topic跟Broker的映射关系。
+ 收发消息前，先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，也可以在发送消息时自动创建Topic。
+ Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息。
+ Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取当前订阅Topic存在哪些Broker上，然后直接跟Broker建立连接通道，开始消费消息。





## 源码解析

