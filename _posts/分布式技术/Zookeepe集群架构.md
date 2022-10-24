---
layout: post
title: Zookeeper入门指南
categories: [分布式, zookeeper]
description: Zookeeper
keywords: Zookeeper, 分布式
---

## 前言

同学们，在上一章中，我们主要讲了Zookeeper两种启动模式以及具体如何搭建。本章内容主要讲的是集群相关的原理内容，第一章可以当做是Zookeeper原理篇的基础部分，本章则是Zookeeper原理篇进阶部分，有关于Zookeeper集群的读写机制、ZAB协议的知识解析。

**本篇的内容主要包含以下几点：**

- **Zookeeper 集群架构**
- **Zookeeper 读写机制**
- **ZAB协议**
- 关于Zookeeper 集群的一些其他讨论
  - **Zookeeper（读性能）可伸缩性 和 Observer节点**
  - **Zookeeper 与 CAP 理论**
  - **Zookeeper 作为 服务注册中心的局限性**

## 一、Zookeeper 集群架构

接下来我们来说一说Zookeeper的集群架构。

![](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzczOC8xNzkxZjY4ZDkxZDMwZjVlNzg3MTc5YjE2OWQzOThlYS5wbmc=)

![img](https://imgser.wangyeyixia.com/aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzczOC8xNzkxZjY4ZDkxZDMwZjVlNzg3MTc5YjE2OWQzOThlYS5wbmc%3D.png?w=700&webp=1)

### Zookeeper 集群中的角色

> 第一章提过，Zookeeper中，**能改变ZooKeeper服务器状态的操作称为事务操作。一般包括数据节点创建与删除、数据内容更新和客户端会话创建与失效等操作**。

- **Leader 领导者** ：Leader 节点负责Zookeeper集群内部投票的发起和决议（一次事务操作），更新系统的状态；同时它也能接收并且响应Client端发送的请求。
- Learner 学习者
  - **Follower 跟随者** ： Follower 节点用于接收并且响应Client端的请求，如果是事务操作，会将请求转发给Leader节点，发起投票，参与集群的内部投票，
  - **Observer 观察者**：Observer 节点功能和Follower相同，只是Observer 节点不参与投票过程，只会同步Leader节点的状态。
- **Client 客户端**

Zookeeper 通过**复制**来实现**高可用**。在上一章提到的集群模式（replicated mode）下，以`Leader`节点为准，Zookeeper的`ZNode`树上面的每一个修改都会被同步（复制）到其他的Server 节点上面。

> 上面实际上只是一个概念性的简单叙述，在看完下文的**读写机制**和**ZAB协议的两种模式**之后，你就会对这几种角色有一个更加深刻的认识。

## 二、Zookeeper 读写机制

### 读写流程

下图就是集群模式下一个Zookeeper Server节点提供读写服务的一个流程。

![Zookeeper学习系列【三】Zookeeper 集群架构、读写机制以及一致性原理(ZAB协议)](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzQyMS9hZDI0MjFjMjllOTY1NjI0OWI5OWM1OWJhOGRhNWRiNS5wbmc=)

image

如上图所示，每个Zookeeper Server节点除了包含一个请求处理器来处理请求以外，都会有一个**内存数据库(ReplicatedDatabase) \**用于持久化数据。\**ReplicatedDatabase** 包含了整个Data Tree。

来自于Client的读服务（Read Requst），是直接由对应Server的本地副本来进行服务的。

至于来自于Client的写服务（Write Requst），因为Zookeeper要保证每台Server的本地副本是一致的（单一系统映像），需要通过一致性协议（后文提到的ZAB协议）来处理，成功处理的写请求（数据更新）会先序列化到每个Server节点的本地磁盘（为了再次启动的数据恢复）再保存到内存数据库中。

![Zookeeper学习系列【三】Zookeeper 集群架构、读写机制以及一致性原理(ZAB协议)](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzM0OC8xOGQ5ZjkyNDIxNGE4YTJjNWQxNGJkZjRhYTQ4NDM0NC5wbmc=)

image

集群模式下，Zookeeper使用简单的同步策略，通过以下三条基本保证来实现**数据的一致性**：

- 全局**串行化**所有的**写操作**

  > 串行化可以把变量包括对象,转化成连续bytes数据. 你可以将串行化后的变量存在一个文件里或在网络上传输. 然后再反串行化还原为原来的数据。

- 保证**同一客户**端的指令被FIFO执行（以及消息通知的FIFO）

> FIFO -先入先出

- 自定义的原子性消息协议

  简单来说，对数据的写请求，都会被转发到Leader节点来处理，Leader节点会对这次的更新发起投票，并且发送提议消息给集群中的其他节点，当半数以上的Follower节点将本次修改持久化之后，Leader 节点会认为这次写请求处理成功了，提交本次的事务。

### 乐观锁

Zookeeper 的核心思想就是，提供一个**非锁机制的Wait Free 的用于分布式系统同步的核心服务**。其核心对于文件、数据的读写服务，并**不提供加锁互斥的服务**。

但是由于Zookeeper的每次更新操作都会更新ZNode的版本（详见第一章），也就是客户端可以自己基于版本的对比，来实现更新数据时的加锁逻辑。例如下图。

![Zookeeper学习系列【三】Zookeeper 集群架构、读写机制以及一致性原理(ZAB协议)](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzM3My9lZWFhOTI0NzkwNWRlOWE3OGNiYzgxYjZlM2UxZGZlZC5wbmc=)

image

就像我们更新数据库时，会新增一个version字段，通过更新前后的版本对比来实现乐观锁。

## 三、ZAB协议

终于到了ZAB协议，讲述完ZAB协议，大家对Zookeeper的一些特性会有更深的体会，对本文的其他内容也会有更透彻的理解。

ZAB 协议是为分布式协调服务ZooKeeper专门设计的一种**支持崩溃恢复**的**一致性协议**，这个机制保证了各个server之间的同步。全称 Zookeeper Atomic Broadcast Protocol - Zookeeper 原子广播协议。

### 两种模式

Zab协议有两种模式，它们分别是**恢复模式**和**广播模式**。

广播模式

广播模式类似于分布式事务中的 Two-phase commit （两阶段式提交），因为Zookeeper中一次写操作就是被当做一个事务，所以这实际上本质是相同的。

![Zookeeper学习系列【三】Zookeeper 集群架构、读写机制以及一致性原理(ZAB协议)](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzk0OC82ZGU2MzI0MmZhNDAwMzdiZTkxMWUxMzE0NmQ3YTE0NC5wbmc=)

image

在**广播模式**，一次写请求要经历以下的步骤

1. ZooKeeper Server接受到Client的写请求
2. 写请求都被转发给`Leader`节点
3. `Leader`节点先将更新持久化到本地
4. `Leader`节点将此次更新提议(propose)给`Followers`，进入收集选票的流程
5. `Follower`节点接收请求，成功将修改持久化到本地，发送一个ACK给`Leader`
6. `Leader`接收到半数以上的ACK时，`Leader`将广播commit消息并在本地deliver该消息。
7. 当收到`Leader`发来的commit消息时，`Follower`也会deliver该消息。

广播协议在所有的通讯过程中使用TCP的FIFO信道，通过使用该信道，使保持有序性变得非常的容易。通过FIFO信道，消息被有序的deliver。只要收到的消息一被处理，其顺序就会被保存下来。

但是这种模式下，如果`Leader`自身发生了故障，Zookeeper的集群不就提供不了写服务了吗？这就引入了下面的恢复模式。

恢复模式

简单点来说，当集群中的**`Leader`故障**或者**服务启动**的时候，ZAB就会进入恢复模式，其中包括`Leader`选举和完成其他Server和`Leader`之间的**状态同步**。

> *NOTE：选主是ZAB协议中最为重要和复杂的过程。这里面的概念知识较多，放在本章一起讲反而不利于理解本章的知识，所以我打算在下一章单独介绍，同学们可以选择性地食用。*

## 关于Zookeeper 集群的一些其他讨论

### 1. Zookeeper（读性能）可伸缩性 和 Observer节点

一个集群的可伸缩性即可以引入更多的集群节点，来提升某种性能。Zookeeper实际上就是提供读服务和写服务。在最早的时候，Zookeeper是通过引入`Follower`节点来提升**读服务**的性能。但是根据我们之前学习过的读写机制和ZAB协议的内容，引入新的`Follower`节点，会造成Zookeeper 写服务的下降，因为`Leader`发起的投票是要半数以上的`Follower`节点响应才会成功，你`Follower`多了，就增加了协议中投票过程的压力，可能会拖慢整个投票响应的速度。结果就是，**`Follower`节点增加，集群的写操作吞吐会下降**。

在这种情况下，Zookeeper 在3.3.3版本之后，在集群架构中引入了`Observer`角色，和`Follower`唯一的区别的就是不参与投票不参与选主。这样既提升了读性能，又不会影响写性能。

另外提一句，Zookeeper的写性能是不能被扩展的，这也是他不适合当做服务注册发现中心的一个原因之一，在服务发现和健康监测场景下，随着服务规模的增大，无论是应用频繁发布时的服务注册带来的写请求，还是刷毫秒级的服务健康状态带来的写请求，都会Zookeeper带来很大的写压力，因为它本身的写性能是无法扩展的。后文引的文章会详细介绍。

### 2. Zookeeper 与 CAP 理论

分布式领域中存在CAP理论：

- **C：Consistency**，一致性，数据一致更新，所有数据变动都是同步的。
- **A：Availability**，可用性，系统具有好的响应性能。
- **P：Partition tolerance**，分区容错性。以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择，也就是说无论任何消息丢失，系统都可用。

> [CAP 定理的含义 -- 阮一峰](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2018%2F07%2Fcap.html)

该理论已被**证明**：任何分布式系统只可同时满足两点，无法三者兼顾。 因此，将精力浪费在思考如何设计能满足三者的完美系统上是愚钝的，应该根据应用场景进行适当取舍。

根据我们前面学习过的读写机制和ZAB协议的内容，Zookeeper本质应该是一个偏向CP的分布式系统。因为广播协议本质上是牺牲了系统的响应性能的。另外从它的以下几个特点也可以看出。也就是在第一章最后提出的几个特点。

> ① 顺序一致性
> 从同一个客户端发起的事务请求，最终将会严格按照其发起顺序被应用到ZooKeeper中。

> ② 原子性
> 所有事务请求的结果在集群中所有机器上的应用情况是一致的，也就是说要么整个集群所有集群都成功应用了某一个事务，要么都没有应用，一定不会出现集群中部分机器应用了该事务，而另外一部分没有应用的情况。

> ③ 单一视图
> 无论客户端连接的是哪个ZooKeeper服务器，其看到的服务端数据模型都是一致的。

> ④ 可靠性
> 一旦服务端成功地应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会被一直保留下来，除非有另一个事务又对其进行了变更。

### 3. Zookeeper 作为 服务注册中心的局限性

直接引一篇阿里中间件的文章吧，讲的比我好。实际在生产情况下，大多数公司没有达到像大公司那样的微服务量级，Zookeeper是完全能满足服务注册中心的需求的。

> [阿里巴巴为什么不用 ZooKeeper 做服务发现？](https://links.jianshu.com/go?to=http%3A%2F%2Fjm.taobao.org%2F2018%2F06%2F13%2F%E5%81%9A%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%EF%BC%9F%2F)

## 总结

本章主要介绍了Zookeeper的集群架构，详述了ZK的几种角色和组件，还介绍了Zookeeper的读写机制和最核心的ZAB协议，最后对其他一些比较杂的知识点统一归在一起讨论了一下。

本章的知识我本人认为信息量还是蛮大的，整理完之后我自己对Zookeeper集群服务的机制原理有了更深的体会。阅读时***能够结合第一章的一些基础概念，这样更有助于理解，让知识点前后呼应。希望能对你理解Zookeeper起到帮助。

下一章我会详细介绍本章未介绍的Zookeeper选主过程(Leader Election)。

## 参考

[1] <[https://zookeeper.apache.org/doc/r3.4.13/zookeeperInternals.html#sc_atomicBroadcast](https://links.jianshu.com/go?to=https%3A%2F%2Fzookeeper.apache.org%2Fdoc%2Fr3.4.13%2FzookeeperInternals.html%23sc_atomicBroadcast) 官方文档

[2] [阿里巴巴为什么不用 ZooKeeper 做服务发现？](https://links.jianshu.com/go?to=http%3A%2F%2Fjm.taobao.org%2F2018%2F06%2F13%2F%E5%81%9A%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%EF%BC%9F%2F)

[3] [https://www.cnblogs.com/sunddenly/p/4138580.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fsunddenly%2Fp%2F4138580.html)

[4] [CAP 定理的含义 -- 阮一峰](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2018%2F07%2Fcap.html)
