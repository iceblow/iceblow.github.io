---
layout: post
title: Springcloud Gateway
categories: [架构]
description: Springcloud Gateway
keywords: gateway, 网关, Springcloud, 微服务
---

Spring Cloud Gateway 旨在提供一种简单而有效的方式来路由到 API，并为它们提供横切关注点，例如：安全性、监控/指标和弹性。

主要特性：

- 基于 Spring Framework 5、Project Reactor 和 Spring Boot 2.0 构建
- 能够匹配任何请求属性的路由。
- 断言和过滤器。
- 断路器集成。
- Spring Cloud Discovery客户端集成
- 易于编写断言和过滤器
- 请求速率限制
- 路径重写

#### 核心概念

- **路由**（route） ：路由信息的组成：由一个ID、一个目的URL、一组断言工厂、一组Filter组成。如果路由断言为真，说明请求URL和配置路由匹配。
- **断言**（Predicate）：ServerWebExchange。Spring Cloud Gateway的断言函数允许开发者去定义匹配来自于Http Request中的任何信息比如请求头和参数。
- **过滤器**（Filter）： 一个标准的Spring WebFilter。 Spring Cloud Gateway中的Filter分为两种类型的Filter，分别是Gateway Filter和Global Filter。过滤器Filter将会对请求和响应进行修改处理。

以下是工作原理

<img src="https://cdn.jsdelivr.net/gh/iceblow/images/20220301105532.png" alt="Spring Cloud 网关示意图" style="zoom: 67%;" />

客户端向 Spring Cloud Gateway 发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关 Web 处理程序。

web处理程序通过特定于请求的过滤器链运行请求。过滤器用虚线划分的原因是过滤器可以在发送代理请求之前和之后运行逻辑。执行所有“预”过滤器逻辑。然后发出代理请求。发出代理请求后，将运行“发布”过滤器逻辑。

#### GatewayFilter

路由过滤器允许以某种方式修改传入的 HTTP 请求或传出的 HTTP 响应。路由过滤器的范围是特定的路由。Spring Cloud Gateway 包含许多内置的 GatewayFilter 工厂。

##### 断路器CircuitBreaker GatewayFilter

Spring Cloud CircuitBreaker GatewayFilter 工厂使用 Spring Cloud CircuitBreaker API 将网关路由包装在断路器中。Spring Cloud CircuitBreaker 支持多个可与 Spring Cloud Gateway 一起使用的库。Spring Cloud 开箱即用地支持 Resilience4J。

##### 限流RequestRateLimiter` `GatewayFilter

`RequestRateLimiter` `GatewayFilter`工厂使用实现`RateLimiter`来确定是否允许当前请求继续。如果不是，`HTTP 429 - Too Many Requests`则返回（默认情况下）状态。

#### GlobalFilter

当请求与路由匹配时，过滤 Web 处理程序会将 的所有实例`GlobalFilter`和所有特定于路由的实例添加`GatewayFilter`到过滤器链中。这个组合的过滤器链是按`org.springframework.core.Ordered`接口排序的，你可以通过实现`getOrder()`方法来设置。

##### 负载均衡LoadBalancerClient

在`LoadBalancerClientFilter`名为 的交换属性中查找 URI `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`。如果 URL 有一个方案`lb`（例如`lb://myservice`），它使用 Spring Cloud`LoadBalancerClient`将名称（在这种情况下）解析为`myservice`实际的主机和端口，并替换同一属性中的 URI。



