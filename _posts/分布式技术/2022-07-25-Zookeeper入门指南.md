---
layout: post
title: Zookeeper入门指南
categories: [分布式, zookeeper]
description: Zookeeper
keywords: Zookeeper, 分布式
---

本文档包含帮助您快速开始使用 ZooKeeper 的信息，包含单个 ZooKeeper 服务器的简单安装说明、一些用于验证它是否正在运行的命令以及一个简单的编程示例。最后，为方便起见，还有一些关于更复杂安装的部分，例如运行复制部署和优化事务日志。

### 下载

要获得 ZooKeeper 发行版，请从 Apache 下载镜像之一下载最近的[稳定](http://zookeeper.apache.org/releases.html)版本。

### 单机模式

载了一个稳定的 ZooKeeper 版本，解压它并 cd 到根目录

要启动 ZooKeeper，您需要一个配置文件。这是一个示例，在**conf/zoo.cfg**中创建它：

```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

更改**dataDir**的值以指定现有（开始时为空）目录。以下是每个字段的含义：

- **tickTime\**：ZooKeeper 使用的基本时间单位，以毫秒为单位。它用于做心跳，最小会话超时将是 tickTime 的两倍。
- **dataDir\**：存储内存数据库快照的位置，除非另有说明，否则存储数据库更新的事务日志。
- **clientPort\**：监听客户端连接的端口

现在您已经创建了配置文件，您可以启动 ZooKeeper：

```sh
bin/zkServer.sh start
```

ZooKeeper 使用*logback记录消息——程序员指南的*[日志](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#Logging)部分提供了更多详细信息。根据 logback 配置，您将看到到达控制台（默认）和/或日志文件的日志消息。

此处概述的步骤以单机模式运行 ZooKeeper。没有复制，因此如果 ZooKeeper 进程失败，服务将关闭。这对于大多数开发情况都很好，但是要在复制模式下运行 ZooKeeper，请参阅[Running Replicated ZooKeeper](https://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_RunningReplicatedZooKeeper)。

### 管理 ZooKeeper 存储

对于长期运行的生产系统，ZooKeeper 存储必须在外部进行管理（dataDir 和日志）。有关详细信息，请参阅[维护部分。](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance)

### 连接到 ZooKeeper

```sh
$ bin/zkCli.sh -server 127.0.0.1:2181
```

这使您可以执行简单的类似文件的操作。

连接后，您应该会看到如下内容：

```sh
Connecting to localhost:2181
...
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
```

在 shell 中，键入`help`以获取可以从客户端执行的命令列表，如下所示：

```sh
[zkshell: 0] help
ZooKeeper -server host:port cmd args
addauth scheme auth
close
config [-c] [-w] [-s]
connect host:port
create [-s] [-e] [-c] [-t ttl] path [data] [acl]
delete [-v version] path
deleteall path
delquota [-n|-b] path
get [-s] [-w] path
getAcl [-s] path
getAllChildrenNumber path
getEphemerals path
history
listquota path
ls [-s] [-w] [-R] path
ls2 path [watch]
printwatches on|off
quit
reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
redo cmdno
removewatches path [-c|-d|-a] [-l]
rmr path
set [-s] [-v version] path data
setAcl [-s] [-v version] [-R] path acl
setquota -n|-b val path
stat [-w] path
sync path
```

从这里，您可以尝试一些简单的命令来感受这个简单的命令行界面。首先，首先发出 list 命令，如 中`ls`，产生：

```sh
[zkshell: 8] ls /
[zookeeper]
```

接下来，通过运行创建一个新的 znode `create /zk_test my_data`。这将创建一个新的 znode 并将字符串“my_data”与该节点相关联。你应该看到：

```sh
[zkshell: 9] create /zk_test my_data
Created /zk_test
```

发出另一个`ls /`命令来查看目录的样子：

```sh
[zkshell: 11] ls /
[zookeeper, zk_test]
```

请注意，现在已经创建了 zk_test 目录。

`get`接下来，通过运行命令验证数据是否与 znode 关联，如下所示：

```sh
[zkshell: 12] get /zk_test
my_data
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 5
mtime = Fri Jun 05 13:57:06 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0
dataLength = 7
numChildren = 0
```

`set`我们可以通过发出命令来更改与 zk_test 关联的数据，如下所示：

```sh
zkshell: 14] set /zk_test junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
[zkshell: 15] get /zk_test
junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
```

最后，让我们`delete`通过发出：

```sh
[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
```

### ZooKeeper 编程

ZooKeeper 具有 Java 绑定和 C 绑定。它们在功能上是等效的。C 绑定存在两种变体：单线程和多线程。这些仅在消息传递循环的完成方式上有所不同。有关更多信息，请参阅[ZooKeeper 程序员指南中的编程示例以](https://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#ch_programStructureWithExample)获取使用不同 API 的示例代码。

### 运行复制的 ZooKeeper

以单机模式运行 ZooKeeper 便于评估、一些开发和测试。但在生产中，您应该以复制模式运行 ZooKeeper。同一应用程序中的一组复制服务器称为*仲裁*，在复制模式下，仲裁中的所有服务器都具有相同配置文件的副本。

> 对于复制模式，至少需要三台服务器，强烈建议您使用奇数台服务器。如果您只有两台服务器，那么您的情况是，如果其中一台出现故障，则没有足够的机器来形成多数仲裁。两台服务器本质上**不如**一台服务器稳定，因为有两个单点故障。

复制模式所需的**conf/zoo.cfg**文件类似于独立模式中使用的文件，但有一些不同之处。这是一个例子：

```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```

新条目**initLimit**是 ZooKeeper 用来限制法定人数中的 ZooKeeper 服务器必须连接到领导者的时间长度。条目**syncLimit**限制服务器与领导者之间的过期时间。

对于这两种超时，您可以使用**tickTime**指定时间单位。在此示例中，initLimit 的超时时间为 5 个滴答声，每滴答声为 2000 毫秒，即 10 秒。

表单*server.X*的条目列出了组成 ZooKeeper 服务的服务器。当服务器启动时，它通过在数据目录中查找文件*myid来知道它是哪个服务器。*该文件包含 ASCII 格式的服务器编号。

最后，请注意每个服务器名称后面的两个端口号：“2888”和“3888”。对等点使用前一个端口连接到其他对等点。这样的连接是必要的，以便对等方可以进行通信，例如，就更新顺序达成一致。更具体地说，ZooKeeper 服务器使用此端口将追随者连接到领导者。当一个新的领导者出现时，一个追随者使用这个端口打开一个到领导者的 TCP 连接。因为默认的leader选举也使用TCP，所以我们目前需要另外一个端口来进行leader选举。这是服务器条目中的第二个端口。

> 如果您想在一台机器上测试多个服务器，请将服务器名称指定为具有唯一仲裁和领导选举端口（即上面示例中的 2888: *3888、2889* :3889、2890:3890）的服务器名称。配置文件。当然，单独的 _dataDir_s 和不同的 _clientPort_s 也是必要的（在上面复制的示例中，在单个*localhost*上运行，您仍然会有三个配置文件）。
>
> 请注意，在一台机器上设置多个服务器不会产生任何冗余。如果发生了导致机器死机的事情，所有的 zookeeper 服务器都将处于脱机状态。完全冗余要求每台服务器都有自己的机器。它必须是完全独立的物理服务器。同一物理主机上的多个虚拟机仍然容易受到该主机完全故障的影响。
>
> 如果您的 ZooKeeper 机器中有多个网络接口，您还可以指示 ZooKeeper 绑定所有接口，并在网络故障时自动切换到健康的接口。有关详细信息，请参阅[配置参数](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#id_multi_address)。

### 其他优化

还有一些其他配置参数可以大大提高性能：

为了降低更新延迟，拥有一个专门的事务日志目录很重要。默认情况下，事务日志与数据快照和*myid*文件放在同一目录中。dataLogDir 参数指示用于事务日志的不同目录。
