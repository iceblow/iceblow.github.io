---
layout: post
title: Spring Cloud Alibaba
categories: 框架
description: 分布式框架
keywords: 分布式框架
excerpt: Spring Cloud Alibaba 为分布式应用开发提供一站式解决方案。它包含开发分布式应用程序所需的所有组件，使您可以轻松地使用 Spring Cloud 开发应用程序。
---

Spring Cloud Alibaba 为分布式应用开发提供一站式解决方案。它包含开发分布式应用程序所需的所有组件，使您可以轻松地使用 Spring Cloud 开发应用程序。

使用Spring Cloud Alibaba，您只需添加一些注解和少量配置，即可将Spring Cloud应用连接到阿里巴巴的分布式解决方案，并通过阿里巴巴中间件构建分布式应用系统。

[详细介绍](https://spring.io/projects/spring-cloud-alibaba)

### 对比Springcloud

| 功能组件                                             | Spring Cloud                         | Dubbo Spring Cloud                                   |
| ---------------------------------------------------- | ------------------------------------ | ---------------------------------------------------- |
| 分布式配置（Distributed configuration）              | Git、Zookeeper、Consul、JDBC         | Spring Cloud 分布式配置 + Dubbo 配置中心[6]          |
| 服务注册与发现（Service registration and discovery） | Eureka、Zookeeper、Consul            | Spring Cloud 原生注册中心[7] + Dubbo 原生注册中心[8] |
| 负载均衡（Load balancing）                           | Ribbon（随机、轮询等算法）           | Dubbo 内建实现（随机、轮询等算法 + 权重等特性）      |
| 服务熔断（Circuit Breakers）                         | Spring Cloud Hystrix                 | Spring Cloud Hystrix + Alibaba Sentinel[9] 等        |
| 服务调用（Service-to-service calls）                 | Open Feign、`RestTemplate`           | Spring Cloud 服务调用 + Dubbo `@Reference`           |
| 链路跟踪（Tracing）                                  | Spring Cloud Sleuth[10] + Zipkin[11] | Zipkin、opentracing 等                               |

### 组件

#### 流量控制和服务降级

使用[Alibaba Sentinel](https://github.com/alibaba/Sentinel/)流量控制、断路和系统自适应保护

#### 服务注册与发现

实例可以注册到[阿里巴巴 Nacos](https://github.com/alibaba/nacos/)，客户端可以使用 Spring 管理的 bean 来发现实例。通过 Spring Cloud Netflix 支持 Ribbon，客户端负载均衡器

#### 分布式配置

使用[阿里巴巴Nacos](https://github.com/alibaba/nacos/)作为数据存储

#### 事件驱动

构建与 Spring Cloud Stream [RocketMQ](https://rocketmq.apache.org/) Binder连接的高度可扩展的事件驱动微服务

#### 消息总线

使用 Spring Cloud Bus RocketMQ 链接分布式系统的节点

#### 分布式事务

支持高性能、易用的分布式事务解决方案[Seata](https://github.com/seata/seata)

#### Dubbo RPC

通过[Apache Dubbo RPC](https://dubbo.apache.org/en-us/)扩展Spring Cloud服务到服务调用的通信协议

#### Spring Boot Starter

- 对象存储OSS服务
- 短信服务
- Redis
- MySQL







