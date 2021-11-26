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

zookeeper安装过程省略（下载地址https://downloads.apache.org/zookeeper/stable/，包名选择带bin的）。

客户端连接可以使用zk原生的api，也可以使用curator连接。Curator 是 Netflix 公司开源的一套 zookeeper 客户端框架，解决了很多 Zookeeper 客户端非常底层的细节开发工作，包括连接重连、反复注册 Watcher 和 NodeExistsException 异常等。

Curator 包含了几个包：

- **curator-framework**：对 zookeeper 的底层 api 的一些封装。
- **curator-client**：提供一些客户端的操作，例如重试策略等。
- **curator-recipes**：封装了一些高级特性，如：Cache 事件监听、选举、分布式锁、分布式计数器、分布式 Barrier 等。

```java
public class CuratorDemo {
    public static void main(String[] args) throws Exception {
        // 连接地址,可以多个,拼接
        String zookeeperConnectionString = "localhost:2181";
        // 重试策略
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);

        // 客户端
        CuratorFramework client = CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy);
        client.start();

        String data = "myData";
        // 创建节点,并返回节点目录
        String result = client.create().forPath("/node1", data.getBytes());
      	// 结果：/node1
        System.err.println("result = " + result);

        // 关闭客户端
        client.close();
    }
}
```

### 客户端命令

在 zookeeper 中，可以说 zookeeper 中的所有存储的数据是由 znode 组成的，节点也称为 znode，并以 key/value 形式存储数据。

整体结构类似于 linux 文件系统的模式以树形结构存储。其中根路径以 **/** 开头。

进入 zookeeper 安装的 bin 目录，通过sh zkCli.sh打开命令行终端。

#### ls 命令

ls 命令用于查看某个路径下目录列表。

```shell
ls path
```

- path：代表路径。

案例：

```shell
[zk: localhost:2181(CONNECTED) 17] ls /
[node, node-1, node1, node2, test, zookeeper]
```

除了zookeeper节点外，其他几个节点都是我们测试创建的。

#### get 命令

get 命令用于获取节点数据和状态信息。

```shell
get path [watch]
```

- path：代表路径。
- [watch]：对节点进行事件监听。

案例：

```shell
[zk: localhost:2181(CONNECTED) 18] get /node1
myData
```

#### stat 命令

stat 命令用于查看节点状态信息。

格式：

```shell
stat path [watch]
```

- path：代表路径。
- [watch]：对节点进行事件监听。

案例：

```shell
[zk: localhost:2181(CONNECTED) 22] stat /node1
cZxid = 0x13
ctime = Thu Nov 25 16:39:01 CST 2021
mZxid = 0x13
mtime = Thu Nov 25 16:39:01 CST 2021
pZxid = 0x13
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

#### create 命令

create 命令用于创建节点并赋值。

格式：

```
create [-s] [-e] path data acl
```

- **[-s] [-e]**：-s 和 -e 都是可选的，-s 代表顺序节点， -e 代表临时节点，注意其中 -s 和 -e 可以同时使用的，并且临时节点不能再创建子节点。
- **path**：指定要创建节点的路径，比如 **/runoob**。
- **data**：要在此节点存储的数据。
- **acl**：访问权限相关，默认是 world，相当于全世界都能访问。

#### set 命令

set 命令用于修改节点存储的数据。

格式：

```
set path data [version]
```

- **path**：节点路径。
- **data**：需要存储的数据。
- **[version]**：可选项，版本号(可用作乐观锁)。

### 节点特性

1、同一级节点 key 名称是唯一的

2、创建节点时，必须要带上全路径

3、session 关闭，临时节点清除

4、自动创建顺序节点

5、watch 机制，监听节点变化

6、delete 命令只能一层一层删除

 有了上述众多节点特性，使得 zookeeper 能开发不出不同的经典应用场景，比如：

- 数据发布/订阅
- 负载均衡
- 分布式协调/通知
- 集群管理
- master 管理
- 分布式锁
- 分布式队列

### 分布式锁实现

分布式锁是控制分布式系统之间同步访问共享资源的一种方式。

下面介绍 zookeeper 如何实现分布式锁，讲解排他锁和共享锁两类分布式锁。

#### 排他锁

排他锁（Exclusive Locks），又被称为写锁或独占锁，如果事务T1对数据对象O1加上排他锁，那么整个加锁期间，只允许事务T1对O1进行读取和更新操作，其他任何事务都不能进行读或写。

定义锁：

```
/exclusive_lock/lock
```

**实现方式：**

利用 zookeeper 的同级节点的唯一性特性，在需要获取排他锁时，所有的客户端试图通过调用 create() 接口，在 **/exclusive_lock** 节点下创建临时子节点 **/exclusive_lock/lock**，最终只有一个客户端能创建成功，那么此客户端就获得了分布式锁。同时，所有没有获取到锁的客户端可以在 **/exclusive_lock** 节点上注册一个子节点变更的 watcher 监听事件，以便重新争取获得锁。

#### 共享锁

共享锁（Shared Locks），又称读锁。如果事务T1对数据对象O1加上了共享锁，那么当前事务只能对O1进行读取操作，其他事务也只能对这个数据对象加共享锁，直到该数据对象上的所有共享锁都释放。

定义锁:

```
/shared_lock/[hostname]-请求类型W/R-序号
```

**实现方式：**

1、客户端调用 create 方法创建类似定义锁方式的临时顺序节点。

![img](https://gitee.com/dxyin/pic/raw/master/20211125183207.png)

2、客户端调用 getChildren 接口来获取所有已创建的子节点列表。

3、判断是否获得锁，对于读请求如果所有比自己小的子节点都是读请求或者没有比自己序号小的子节点，表明已经成功获取共享锁，同时开始执行度逻辑。对于写请求，如果自己不是序号最小的子节点，那么就进入等待。

4、如果没有获取到共享锁，读请求向比自己序号小的最后一个写请求节点注册 watcher 监听，写请求向比自己序号小的最后一个节点注册watcher 监听。

实际开发过程中，可以 curator 工具包封装的API帮助我们实现分布式锁。

<dependency>  <groupId>org.apache.curator</groupId>  <artifactId>curator-recipes</artifactId>  <version>x.x.x</version> </dependency>

curator 的几种锁方案 ：

- 1、**InterProcessMutex**：分布式可重入排它锁
- 2、**InterProcessSemaphoreMutex**：分布式排它锁
- 3、**InterProcessReadWriteLock**：分布式读写锁

下面例子模拟 20 个线程使用重入排它锁 InterProcessMutex 同时争抢锁：

```java
public class InterprocessLock {

    public static void main(String[] args)  {
        CuratorFramework zkClient = getZkClient();
        String lockPath = "/lock";
        InterProcessMutex lock = new InterProcessMutex(zkClient, lockPath);
        //模拟20个线程抢锁
        for (int i = 0; i < 20; i++) {
            new Thread(new TestThread(i, lock)).start();
        }
    }

    // 创建client
    private static CuratorFramework getZkClient() {
        String zkServerAddress = "localhost:2181";
        ExponentialBackoffRetry retryPolicy = new ExponentialBackoffRetry(1000, 3, 5000);
        CuratorFramework zkClient = CuratorFrameworkFactory.builder()
                .connectString(zkServerAddress)
                .sessionTimeoutMs(5000)
                .connectionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .build();
        zkClient.start();
        return zkClient;
    }

    // 线程类
    static class TestThread implements Runnable {
        private Integer threadFlag;
        private InterProcessMutex lock;

        public TestThread(Integer threadFlag, InterProcessMutex lock) {
            this.threadFlag = threadFlag;
            this.lock = lock;
        }

        @Override
        public void run() {
            try {
                lock.acquire();
                System.out.println("第"+threadFlag+"线程获取到了锁");
                //等到1秒后释放锁
                Thread.sleep(1000);
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                try {
                    lock.release();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

}
```

### 机器注册ID

在我们使用雪花算法的时候，需要配置机器的workId，由于一个服务会部署在多台机器上，这就

造成每个机器的workId是相同的，还是存在雪花算法ID重复的可能。

我们可以利用zookeeper的特性，来生成唯一的workId。

原理：微服务的每个机器连接到zookeeper，创建持久化的顺序节点，每个节点的序号是递增的，不会重复，这样每个机器的workId是唯一的。

雪花算法如果遇到时钟回拨，那么这台机器再注册生成新的节点，新节点和老节点的workId不同，这样可以解决时钟回拨的问题（个人思路）。

demo如下：

```java
package com.example.zookeeper.api;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;

/**
 * 获取注册节点id
 */
public class MachineIdDemo {

    public static void main(String[] args) throws Exception {
        CuratorFramework client = createClient();
        String path = "/tem_seq";

        // PERSISTENT_SEQUENTIAL:持久化序列;  EPHEMERAL_SEQUENTIAL:短暂序列(会自动删除)
        for (int i = 0; i < 4; i++) {
            // 返回节点名称,包含序号
            String result = client.create().withMode(CreateMode.PERSISTENT_SEQUENTIAL).forPath(path, "seq_test".getBytes());
            System.err.println(i + "节点:" + result);
        }
        client.close();
    }

    public static CuratorFramework createClient() {
        String zookeeperConnectionString = "localhost:2181";
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        CuratorFramework client = CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy);
        client.start();
        return client;
    }
}

```

打印日志结果：

```
0节点:/tem_seq0000000011
1节点:/tem_seq0000000012
2节点:/tem_seq0000000013
3节点:/tem_seq0000000014
```

对节点名称处理后，可以得到workId。

