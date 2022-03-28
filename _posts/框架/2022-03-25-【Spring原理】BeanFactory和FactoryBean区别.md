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

一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的。

Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。

这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean。

```java
public interface FactoryBean<T> {

	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

  // 返回FactoryBean创建的bean
	@Nullable
	T getObject() throws Exception;

  // 返回创建的bean类型
	@Nullable
	Class<?> getObjectType();

  // 默认单例
	default boolean isSingleton() {
		return true;
	}
}
```

### 两者对比

| 对比     | BeanFactory                                                  | FactoryBean<T>                                               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 类型     | 接口                                                         | 接口                                                         |
| 作用     | 生产bean的工厂，基础型IoC容器                                | 能生产或者修饰对象生成的工厂Bean，实现该接口可以定制实例化bean |
| 主要方法 | getBean(String name)<br/>getBean(Class<T> requiredType)<br/>isSingleton(String name) | getObject()<br/>getObjectType()<br/>isSingleton()            |

