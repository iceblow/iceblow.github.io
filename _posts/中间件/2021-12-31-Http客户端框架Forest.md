---
layout: post
title: Http客户端框架Forest
categories: [框架, 中间件]
description: some word here
keywords: keyword1, keyword2
---

Forest 是一个开源的 Java HTTP 客户端框架，它能够将 HTTP 的所有请求信息（包括 URL、Header 以及 Body 等信息）绑定到您自定义的 Interface 方法上，能够通过调用本地接口方法的方式发送 HTTP 请求。

### 为什么使用 Forest

使用 Forest 就像使用类似 Dubbo 那样的 RPC 框架一样，只需要定义接口，调用接口即可，不必关心具体发送 HTTP 请求的细节。同时将 HTTP 请求信息与业务代码解耦，方便您统一管理大量 HTTP 的 URL、Header 等信息。而请求的调用方完全不必在意 HTTP 的具体内容，即使该 HTTP 请求信息发生变更，大多数情况也不需要修改调用发送请求的代码。

Forest 不需要您编写具体的 HTTP 调用过程，只需要您定义一个接口，然后通过 Forest 注解将 HTTP 请求的信息添加到接口的方法上即可。请求发送方通过调用您定义的接口便能自动发送请求和接受请求的响应。

### 快速上手

#### maven依赖

若您的项目基于`Spring Boot`，那只要添加下面一个 maven 依赖便可。

```xml
<dependency>
    <groupId>com.dtflys.forest</groupId>
    <artifactId>forest-spring-boot-starter</artifactId>
    <version>1.5.16</version>
</dependency>
```

#### 定义Interface

创建一个`interface`，比如命名为`MyClient`，并创建一个接口方法名为`helloForest`，用`@Request`注解修饰之。

```java

public interface MyClient {

    @Request("http://localhost:8080/hello")
    String helloForest();

}
```

通过`@Request`注解，将上面的`MyClient`接口中的`helloForest()`方法绑定了一个 HTTP 请求， 其 URL 为`http://localhost:8080/hello` ，并默认使用`GET`方式，且将请求响应的数据以`String`的方式返回给调用者。

#### 发送请求

若您已有定义好的 Forest 请求接口(比如名为 `com.yoursite.client.MyClient`)，那就可以开始愉快使用它了。

```java
@Component
public class MyService {
    @Autowired
    private MyClient myClient;

    public void testClient() {
        String result = myClient.helloForest();
        System.out.println(result);
    }

}
```

#### 快捷接口

Forest 是一个以接口 + 注解形式定义请求为主的HTTP框架，发送请求前先需要定义一个 interface 接口类，再组合使用 Forest 的注解（如：`@Get`）定义绑定的HTTP请求参数，随后再实例化接口类对象，最终调用该接口绑定HTTP参数注解的方法实现请求的发送。

这样做有很多优点，比如在 Java 代码和 HTTP 协议之间实现解耦，同时方便众多请求接口的管理。但缺点也很明显：步骤较多，如果想要直接快速的请求一个简单的URL地址就显得过重了。

不过，Forest 自 `1.5.3` 版本起，就提供了快捷接口，不用定义接口类也能发送请求，使用也很方便。

##### 以字符串形式接受响应数据

```java
// Get请求
// 并以 String 类型接受数据
String str = Forest.get("/").executeAsString();
```

##### 以自定义类型形式接受响应数据

```java
// Post请求// 并以自定义的 MyResult 类型接受
MyResult myResult = Forest.post("/").execute(MyResult.class);
```

##### 以带复杂泛型参数的类型形式接受响应数据

```java
// 通过 TypeReference 引用类传递泛型参数
// 就可以将响应数据以带复杂泛型参数的类型接受了
Result<List<User>> userList = Forest.post("/")
        .execute(new TypeReference<Result<List<User>>>() {});
```

##### 异步请求

```java
// 异步 Post 请求
// 通过 onSuccess 回调函数处理请求成功后的结果
// 而 onError 回调函数则在请求失败后被触发
Forest.post("/")
        .async()
        .onSuccess(((data, req, res) -> {
            // data 为响应成功后返回的反序列化过的数据
            // req 为Forest请求对象，即 ForestRequest 类实例
            // res 为Forest响应对象，即 ForestResponse 类实例
        }))
        .onError(((ex, req, res) -> {
            // ex 为请求过程可能抛出的异常对象
            // req 为Forest请求对象，即 ForestRequest 类实例
            // res 为Forest响应对象，即 ForestResponse 类实例
        }))
        .execute();

```

##### 定义请求的各种参数

```java
// 定义各种参数
// 并以 Map 类型接受
Map<String, Object> map = Forest.post("/")
      .backend("okhttp3")        // 设置后端为 okhttp3
      .host("127.0.0.1")         // 设置地址的host为 127.0.0.1
      .port(8080)                // 设置地址的端口为 8080
      .contentTypeJson()         // 设置 Content-Type 头为 application/json
      .addBody("a", 1)           // 添加 Body 项(键值对)： a, 1
      .addBody("b", 2)           // 添加 Body 项(键值对：  b, 2
      .maxRetryCount(3)          // 设置请求最大重试次数为 3
      // 设置 onSuccess 回调函数
      .onSuccess((data, req, res) -> { log.info("success!"); })
      // 设置 onError 回调函数
      .onError((ex, req, res) -> { log.info("error!"); })
      // 设置请求成功判断条件回调函数
      .successWhen((req, res) -> res.noException() && res.statusOk())
      // 执行并返回Map数据类型对象
      .executeAsMap();

```

Forest 的快速接口（如：`Forest.get(String url)`、`Forest.post(String url)`等方法）本质上是返回了一个 Forest 请求对象（即 `ForestRequest` 类对象），Forest 的绝大部分操作都是围绕请求对象所作的工作。

#### 配置

若您的项目依赖`Spring Boot`，并加入了`spring-boot-starter-forest`依赖，就可以通过 `application.yml`/`application.properties` 方式定义配置。

##### 后端 HTTP API

```yaml
forest:
  backend: okhttp3 # 配置后端HTTP API为 okhttp3
```

目前 Forest 支持`okhttp3`和`httpclient`两种后端 HTTP API，若不配置该属性，默认为`okhttp3`. 当然，您也可以改为`httpclient`

##### http配置

```yaml
forest:
  bean-id: config0 # 在spring上下文中bean的id（默认为 forestConfiguration）
  backend: okhttp3 # 后端HTTP框架（默认为 okhttp3）
  max-connections: 1000 # 连接池最大连接数（默认为 500）
  max-route-connections: 500 # 每个路由的最大连接数（默认为 500）
  timeout: 3000 # 请求超时时间，单位为毫秒（默认为 3000）
  connect-timeout: 3000 # 连接超时时间，单位为毫秒（默认为 timeout）
  read-timeout: 3000 # 数据读取超时时间，单位为毫秒（默认为 timeout）
  max-retry-count: 0 # 请求失败后重试次数（默认为 0 次不重试）
  ssl-protocol: SSLv3 # 单向验证的HTTPS的默认SSL协议（默认为 SSLv3）
  logEnabled: true # 打开或关闭日志（默认为 true）
  log-request: true # 打开/关闭Forest请求日志（默认为 true）
  log-response-status: true # 打开/关闭Forest响应状态日志（默认为 true）
  log-response-content: true # 打开/关闭Forest响应内容日志（默认为 false）
```

##### 全局变量定义

Forest 可以在`forest.variables`属性下自定义全局变量。

其中 key 为变量名，value 为变量值。

全局变量可以在任何模板表达式中进行数据绑定。

```yaml
forest:
  variables:
    username: foo
    userpwd: bar
```

### 架构

Forest 会将您定义好的接口通过动态代理的方式生成一个具体的实现类，然后组织、验证 HTTP 请求信息，绑定动态数据，转换数据形式，SSL 验证签名，调用后端 HTTP API(httpclient 等 API)执行实际请求，等待响应，失败重试，转换响应数据到 Java 类型等脏活累活都由这动态代理的实现类给包了。 请求发送方调用这个接口时，实际上就是在调用这个干脏活累活的实现类。

![architecture](https://gitee.com/dxyin/pic/raw/master/20211231170633.svg)



我们讲 HTTP 发送请求的过程分为前端部分和后端部分，Forest 本身是处理前端过程的框架，是对后端 HTTP API 框架的进一步封装。

**前端部分：**

1. Forest 配置： 负责管理 HTTP 发送请求所需的配置。
2. Forest 注解： 用于定义 HTTP 发送请求的所有相关信息，一般定义在 interface 上和其方法上。
3. 动态代理： 用户定义好的 HTTP 请求的`interface`将通过动态代理产生实际执行发送请求过程的代理类。
4. 模板表达式： 模板表达式可以嵌入在几乎所有的 HTTP 请求参数定义中，它能够将用户通过参数或全局变量传入的数据动态绑定到 HTTP 请求信息中。
5. 数据转换： 此模块将字符串数据和`JSON`或`XML`形式数据进行互转。目前 JSON 转换器支持`Jackson`、`Fastjson`、`Gson`三种，XML 支持`JAXB`一种。
6. 拦截器： 用户可以自定义拦截器，拦截指定的一个或一批请求的开始、成功返回数据、失败、完成等生命周期中的各个环节，以插入自定义的逻辑进行处理。
7. 过滤器： 用于动态过滤和处理传入 HTTP 请求的相关数据。
8. SSL： Forest 支持单向和双向验证的 HTTPS 请求，此模块用于处理 SSL 相关协议的内容。

**后端部分：**

后端为实际执行 HTTP 请求发送过程的第三方 HTTP API，目前支持`okHttp3`和`httpclient`两种后端 API。

