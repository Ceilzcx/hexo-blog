---
title: lmax-disruptor
date: 2022-05-25 14:30:22
tags: java
---

## LMAX Disruptor

### 核心概念

+ **Ring Buffer**：version3.0后，环形缓存仅作为存储和更新数据（事件）
+ **Sequence**：序列作为识别组件位置，每个消费者维护一个自己的序列
+ **Sequencer**：序列器接口，有两个实现类（单生产者和多生产者），实现生产者和消费者快速传输的并发算法
+ **Sequence Barrier**：由 `Sequencer` 生产，包含对已发布 `Sequence` 的引用和任何依赖的消费者；也包含是否有数据需要被消费的逻辑
+ **Wait Strategy**：等待策略决定消费者如何等待事件的处理策略
+ **Event**：生产者到消费者传输的数据单位。事件完全由用户决定
+ **Event Processor**：主事件循环处理来自 `disruptor` 的事件，并拥有消费者序列的所有权
+ **Event Handler**：一个由用户实现的接口，作为 `disruptor` 的消费者
+ **Producer**：用户自己的代码调用 `disruptor` 进行排队



