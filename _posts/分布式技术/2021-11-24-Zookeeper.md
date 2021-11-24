---
layout: post
title: Zookeeper
categories: [分布式]
description: Zookeeper
keywords: Zookeeper, 分布式
---

ZooKeeper 是 Apache 软件基金会的一个软件项目，它为大型分布式计算提供开源的**分布式配置服务、同步服务和命名注册**。

ZooKeeper 的架构通过冗余服务实现高可用性。

Zookeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

一个**典型的分布式数据一致性的解决方案**，分布式应用程序可以基于它实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

### zk 数据结构

zookkeeper 提供的名称空间非常类似于标准文件系统，key-value 的形式存储。名称 key 由斜线 **/** 分割的一系列路径元素，zookeeper 名称空间中的每个节点都是由一个路径标识。

![img](https://gitee.com/dxyin/pic/raw/master/20211124163227.jpg)

###  CAP 理论

CAP 理论指出对于一个分布式计算系统来说，不可能同时满足以下三点：

- **一致性（C）**：在分布式环境中，一致性是指数据在多个副本之间是否能够保持一致的特性，等同于所有节点访问同一份最新的数据副本。在一致性的需求下，当一个系统在数据一致的状态下执行更新操作后，应该保证系统的数据仍然处于一致的状态。
- **可用性（A）：**每次请求都能获取到正确的响应，但是不保证获取的数据为最新数据。
- **分区容错性（P）：**分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三项中的两项。

在这三个基本需求中，最多只能同时满足其中的两项，P 是必须的，因此只能在 CP 和 AP 中选择，**zookeeper 保证的是 CP**，对比 spring cloud 系统中的注册中心 eruka 实现的是 AP。

<img src="https://gitee.com/dxyin/pic/raw/master/20211124163638.png" alt="img" style="zoom:67%;" />

### BASE 理论

BASE 是 Basically Available(基本可用)、Soft-state(软状态) 和 Eventually Consistent(最终一致性) 三个短语的缩写。

- **基本可用：**在分布式系统出现故障，允许损失部分可用性（服务降级、页面降级）。
- **软状态：**允许分布式系统出现中间状态。而且中间状态不影响系统的可用性。这里的中间状态是指不同的 data replication（数据备份节点）之间的数据更新可以出现延时的最终一致性。
- **最终一致性：**data replications 经过一段时间达到一致性。

BASE 理论是对 CAP 中的一致性和可用性进行一个权衡的结果，理论的核心思想就是：我们无法做到强一致，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性。

### Java客户端

zookeeper安装过程省略（下载地址https://downloads.apache.org/zookeeper/stable/）。

代码TODO

