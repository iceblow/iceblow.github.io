---
layout: post
title: 【Mybatis系列】Mybatis分页插件Pagehelper
categories: [数据库, 中间件]
description: Mybatis分页说插件Pagehelper
keywords: Mybatis, Pagehelper
---

今天来聊聊Mybatis分页插件Pagehelper，如果使用mybatis持久层框架的话，都会用到该插件。

作为经常使用的分页插件，我们还是要了解其实现原理。

本文从一个Mybatis分页demo入手，结合Pagehelper的多种使用方法，重点关注分页安全调用，最后进行相关源码分析。

### Mybatis分页入门demo

#### 准备

```xml
<dependency>
  <groupId>com.github.pagehelper</groupId>
  <artifactId>pagehelper</artifactId>
  <version>5.1.0</version>
</dependency>
```

mapper.xml

```xml
<select id="list" resultMap="BaseResultMap">
        select 
  				*
        from t_user
</select>
```

dao层，并没有添加page分页参数

```java
List<User> list();
```

#### 测试

我们分页查询t_user表，pageNum = 1 && pageSize = 5

PageHelper.startPage(1, 5)开启分页，代码如下：

```java
@GetMapping("page")
public PageInfo<User> page() {
  
    PageHelper.startPage(1, 5);
    List<User> users = userDao.list();
    PageInfo<User> page = new PageInfo<>(users);
  
    return page;
}
```

结果如下：

```
==>  Preparing: select * from t_user 
==> Parameters: 
<==    Columns: id, username, age
<==        Row: 2, hello, 39
<==        Row: 3, bob, 33
<==        Row: 4, bob, 33
<==        Row: 5, bob, 33
<==        Row: 6, bob, 33
<==        Row: 7, bob, 33
<==        Row: 8, bob, 33
<==        Row: 9, bob, 33
<==        Row: 10, bob, 33
<==        Row: 11, bob, 33
<==        Row: 12, bob, 33
<==        Row: 13, bob, 33
<==      Total: 12
```

全表的数据都查出来了，为啥分页没有生效呢？

这就要谈到Pagehelper的原理了，MyBatis定义了拦截器，Executor 中的 query 方法可以被拦截，com.github.pagehelper.PageInterceptor实现了mybatis的拦截器，因此，我们需要先配置mybatis的插件Pagehelper。

mybatis-config.xml 插件部分代码如下：

```xml
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
    </plugin>
</plugins>
```

```properties
mybatis.config-location=classpath:mybatis-config.xml
```

我们再次执行分页查询后，结果如下：

```
==>  Preparing: SELECT count(0) FROM t_user 
==> Parameters: 
<==    Columns: count(0)
<==        Row: 12
<==      Total: 1

==>  Preparing: select * from t_user LIMIT ? 
==> Parameters: 5(Integer)
<==    Columns: id, username, age
<==        Row: 2, hello, 39
<==        Row: 3, bob, 33
<==        Row: 4, bob, 33
<==        Row: 5, bob, 33
<==        Row: 6, bob, 33
<==      Total: 5
```

可以看到，有两条SQL，第一个是查询count，第二个是查询limit，这样分页查询达到了预期的效果。

### PageHelper使用方法

分页插件支持以下几种调用方式：

#### `PageHelper.startPage` 静态方法调用

除了 `PageHelper.startPage` 方法外，还提供了类似用法的 `PageHelper.offsetPage` 方法。

在你需要进行分页的 MyBatis 查询方法前调用 `PageHelper.startPage` 静态方法即可，紧跟在这个方法后的第一个**MyBatis 查询方法**会被进行分页。

```java
//获取第1页，10条内容，默认查询总数count
PageHelper.startPage(1, 10);
//紧跟着的第一个select方法会被分页
List<Country> list = countryMapper.selectIf(1);
assertEquals(2, list.get(0).getId());
assertEquals(10, list.size());
//分页时，实际返回的结果list类型是Page<E>，如果想取出分页信息，需要强制转换为Page<E>
assertEquals(182, ((Page) list).getTotal());
```

或者直接使用PageInfo包装

```java
//用PageInfo对结果进行包装
PageInfo page = new PageInfo(list);
```

#### RowBounds方式的调用

```java
List<Country> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(1, 10));
```

使用这种调用方式时，你可以使用RowBounds参数进行分页，这种方式侵入性最小，我们可以看到，通过RowBounds方式调用只是使用了这个参数，并没有增加其他任何内容。

分页插件检测到使用了RowBounds参数时，就会对该查询进行**物理分页**。

#### 使用参数方式

想要使用参数方式，需要配置 `supportMethodsArguments` 参数为 `true`，同时要配置 `params` 参数。 例如下面的配置：

```xml
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
        <property name="supportMethodsArguments" value="true"/>
        <property name="params" value="pageNum=pageNumKey;pageSize=pageSizeKey;"/>
	</plugin>
</plugins>
```

在 MyBatis 方法中：

```java
List<Country> selectByPageNumSize(
        @Param("user") User user,
        @Param("pageNumKey") int pageNum,
        @Param("pageSizeKey") int pageSize);
```

当调用这个方法时，由于同时发现了 `pageNumKey` 和 `pageSizeKey` 参数，这个方法就会被分页。params 提供的几个参数都可以这样使用。

### PageHelper安全调用

1. 使用 `RowBounds` 和 `PageRowBounds` 参数方式是极其安全的

2. 使用参数方式是极其安全的

3. 使用 ISelect 接口调用是极其安全的

ISelect 接口方式除了可以保证安全外，还特别实现了将查询转换为单纯的 count 查询方式，这个方法可以将任意的查询方法，变成一个 `select count(*)` 的查询方法。

### 什么时候分页不安全

`PageHelper` 方法使用了静态的 `ThreadLocal` 参数，分页参数和线程是绑定的。

只要你可以保证在 `PageHelper` 方法调用后紧跟 MyBatis 查询方法，这就是安全的。因为 `PageHelper` 在 `finally` 代码段中自动清除了 `ThreadLocal` 存储的对象。

如果代码在进入 `Executor` 前发生异常，就会导致线程不可用，这属于人为的 Bug（例如接口方法和 XML 中的不匹配，导致找不到 `MappedStatement` 时）， 这种情况由于线程不可用，也不会导致 `ThreadLocal` 参数被错误的使用。

但是如果你写出下面这样的代码，就是不安全的用法：

```java
PageHelper.startPage(1, 10);
List<Country> list;
if(param1 != null){
    list = countryMapper.selectIf(param1);
} else {
    list = new ArrayList<Country>();
}
```

这种情况下由于 param1 存在 null 的情况，就会导致 PageHelper 生产了一个分页参数，但是没有被消费，这个参数就会一直保留在这个线程上。当这个线程再次被使用时，就可能导致不该分页的方法去消费这个分页参数，这就产生了莫名其妙的分页。

上面这个代码，应该写成下面这个样子：

```java
List<Country> list;
if(param1 != null){
    PageHelper.startPage(1, 10);
    list = countryMapper.selectIf(param1);
} else {
    list = new ArrayList<Country>();
}
```

### 源码分析

#### PageHelper.startPage()

关键点是把分页参数pageNum和pageSize等放在本地线程的变量中，后面执行查询的会用到。

```java
 /**
     * 开始分页
     *
     * @param pageNum      页码
     * @param pageSize     每页显示数量
     * @param count        是否进行count查询
     * @param reasonable   分页合理化,null时用默认配置
     * @param pageSizeZero true且pageSize=0时返回全部结果，false时分页,null时用默认配置
     */
    public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
        Page<E> page = new Page<E>(pageNum, pageSize, count);
        page.setReasonable(reasonable);
        page.setPageSizeZero(pageSizeZero);
        //当已经执行过orderBy的时候
        Page<E> oldPage = getLocalPage();
        if (oldPage != null && oldPage.isOrderByOnly()) {
            page.setOrderBy(oldPage.getOrderBy());
        }
      // 设置page参数到Threadlocal
        setLocalPage(page);
        return page;
    }
```

```java
protected static final ThreadLocal<Page> LOCAL_PAGE = new ThreadLocal<Page>();

    /**
     * 设置 Page 参数
     *
     * @param page
     */
    protected static void setLocalPage(Page page) {
        LOCAL_PAGE.set(page);
    }
```

#### Mybatis的拦截器

```java
package org.apache.ibatis.plugin;

public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  Object plugin(Object target);

  void setProperties(Properties properties);

}
```

#### PageHelper拦截实现

PageHelper的PageInterceptor实现了以上接口，截取部分代码如下：

```java
 @Override
    public Object intercept(Invocation invocation) throws Throwable {
     				// ......省略部分源码
            //调用方法判断是否需要进行分页，如果不需要，直接返回结果
            if (!dialect.skip(ms, parameter, rowBounds)) {
                //反射获取动态参数
                String msId = ms.getId();
                Configuration configuration = ms.getConfiguration();
                Map<String, Object> additionalParameters = (Map<String, Object>) additionalParametersField.get(boundSql);
                //判断是否需要进行 count 查询
                if (dialect.beforeCount(ms, parameter, rowBounds)) {
                    String countMsId = msId + countSuffix;
                    Long count;
                    //先判断是否存在手写的 count 查询
                    MappedStatement countMs = getExistedMappedStatement(configuration, countMsId);
                    if(countMs != null){
                        count = executeManualCount(executor, countMs, parameter, boundSql, resultHandler);
                    } else {
                        countMs = msCountMap.get(countMsId);
                        //自动创建
                        if (countMs == null) {
                            //根据当前的 ms 创建一个返回值为 Long 类型的 ms
                            countMs = MSUtils.newCountMappedStatement(ms, countMsId);
                            msCountMap.put(countMsId, countMs);
                        }
                        count = executeAutoCount(executor, countMs, parameter, boundSql, rowBounds, resultHandler);
                    }
                    //处理查询总数
                    //返回 true 时继续分页查询，false 时直接返回
                    if (!dialect.afterCount(count, parameter, rowBounds)) {
                        //当查询总数为 0 时，直接返回空的结果
                        return dialect.afterPage(new ArrayList(), parameter, rowBounds);
                    }
                }
                //判断是否需要进行分页查询
                if (dialect.beforePage(ms, parameter, rowBounds)) {
                    //生成分页的缓存 key
                    CacheKey pageKey = cacheKey;
                    //处理参数对象
                    parameter = dialect.processParameterObject(ms, parameter, boundSql, pageKey);
                    //调用方言获取分页 sql
                    String pageSql = dialect.getPageSql(ms, boundSql, parameter, rowBounds, pageKey);
                    BoundSql pageBoundSql = new BoundSql(configuration, pageSql, boundSql.getParameterMappings(), parameter);
                    //设置动态参数
                    for (String key : additionalParameters.keySet()) {
                        pageBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
                    }
                    //执行分页查询
                    resultList = executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, pageKey, pageBoundSql);
                } else {
                    //不执行分页的情况下，也不执行内存分页
                    resultList = executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, cacheKey, boundSql);
                }
            } else {
                //rowBounds用参数值，不使用分页插件处理时，仍然支持默认的内存分页
                resultList = executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
            }
            return dialect.afterPage(resultList, parameter, rowBounds);
        } finally {
            dialect.afterAll();
        }
    }
```

executor.query()执行完成后，finally代码块执行了 dialect.afterAll()，移除了分页参数Page的本地线程变量。

```java
 @Override
    public void afterAll() {
        //这个方法即使不分页也会被执行，所以要判断 null
        AbstractHelperDialect delegate = autoDialect.getDelegate();
        if (delegate != null) {
            delegate.afterAll();
            autoDialect.clearDelegate();
        }
      // 移除ThreadLocal<Page>对象
        clearPage();
    }

		/**
     * 移除本地变量
     */
    public static void clearPage() {
        LOCAL_PAGE.remove();
    }
```

### 最后

PageHelper是Mybatis分页查询的插件，使用物理分页方式，主要原理是Pagehelper在开启分页时把分页参数放在ThreadLocal<Page>中，PageInterceptor拦截了mybatis的Executor 中的 query 方法，然后从ThreadLocal<Page>获取分页参数，执行SQL，默认先查selectCount，再查selectList，最后包装PageInfo。
