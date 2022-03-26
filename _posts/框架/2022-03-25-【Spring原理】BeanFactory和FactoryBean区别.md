---
layout: post
title: 【Spring原理】BeanFactory和FactoryBean区别
categories: [框架]
description: some word here
keywords: keyword1, keyword2
---

在面试中，涉及到Spring原理，经常被问到的一道题就是：BeanFactory和FactoryBean的区别？

我们先来分析BeanFactory。

### BeanFactory

BeanFactory就是生产Bean的工厂，它是基础类型的IoC容器，默认采用延迟初始化策略。

BeanFactory是一个接口，只定义如何访问容器内管理的Bean的方法，它的实现类负责Bean的注册和管理.

```java
public interface BeanFactory {
  // FactoryBean的前缀
	String FACTORY_BEAN_PREFIX = "&";

  // beanName或Class获取bean
	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	boolean containsBean(String name);
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
}
```

### FactoryBean

```java
public interface FactoryBean<T> {

	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

	@Nullable
	T getObject() throws Exception;

	@Nullable
	Class<?> getObjectType();

	default boolean isSingleton() {
		return true;
	}
}
```



### 两者对比

|          | BeanFactory                                                  | FactoryBean<T> |
| -------- | ------------------------------------------------------------ | -------------- |
| 类型     | 接口                                                         | 接口           |
| 作用     | 生产bean的工厂，基础型IoC容器                                |                |
| 主要方法 | getBean(String name)<br/>getBean(Class<T> requiredType)<br/>boolean isSingleton(String name) |                |
|          |                                                              |                |

