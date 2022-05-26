---
layout: post
title: 【JVM系列】JVM调优实战-如何排查内存持续上升
categories: [JVM]
description: JVM调优实战-如何排查内存持续上升
keywords: JVM的内存模型,JVM, 内存模型, 性能调优, 内存
---

我想你肯定遇到过内存溢出，或是内存使用率过高的问题。碰到内存持续上升的情况，其实我们很难从业务日志中查看到具体的问题，那么我们如何进行排查呢？

### top 命令

top 命令是我们在 Linux 下最常用的命令之一，它可以实时显示正在执行进程的 CPU 使用率、内存使用率以及系统负载等信息。其中上半部分显示的是系统的统计信息，下半部分显示的是进程的使用率统计信息。

我这里使用macos，命令和Linux稍有出入，不必在意这些细节。

```shell
top -s 10
```

以上命令代表每10s打印一次系统负载信息。结果如下：

![image-20220517120530321](http://qiniuyun.jeesoul.com/img/20220517120530.png)

我们根据内存进行排序

```shell
top -o mem -s 10
```

![image-20220517121316852](http://qiniuyun.jeesoul.com/img/20220517121316.png)

找到占用内存最大的java进程，pid = 81004

```shell
top -pid 81004
```

![image-20220517121515826](http://qiniuyun.jeesoul.com/img/20220517121515.png)
