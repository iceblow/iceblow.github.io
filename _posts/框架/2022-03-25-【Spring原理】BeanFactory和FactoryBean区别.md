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

### Demo

为了更清晰直观理解BeanFactory和FactoryBean，我们写了一个demo，代码如下：

```java
@Data
@ToString
public class User {

    private String name;

    private String role;
}
```

#### UserFactoryBean

```java
public class UserFactoryBean implements FactoryBean<User> {
	// 定制化bean
    @Override
    public User getObject() throws Exception {
        User user = new User();
        user.setName("UserFactoryBean init name");
        user.setAddr("UserFactoryBean init addr");
        return user;
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

#### 注解@Bean

```java
@Configuration
public class Config {

  // 注解bean
    @Bean
    public User user() {
        User user = new User();
        user.setName("user bean name");
        user.setRole("user bean role");
        return user;
    }
}
```

#### 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {UserFactoryBean.class, Config.class})
@Slf4j
public class UserFactoryBeanTest {

    @Resource
    BeanFactory beanFactory;

    @Test
    public void test() {
        User user = beanFactory.getBean("userFactoryBean", User.class);
        // 结果: User(name=UserFactoryBean init name, role=UserFactoryBean init role)
        log.info(user.toString());

        User user2 = beanFactory.getBean("user", User.class);
        // 结果: User(name=user bean name, role=user bean role)
        log.info(user2.toString());

        Object object = beanFactory.getBean("&userFactoryBean");
        // 结果: true
        System.out.println(object instanceof UserFactoryBean);
    }
}
```

#### 结论

beanFactory从`userFactoryBean`获取到的对象是`getObject()`方法定制的bean，并不是`userFactoryBean`对象本身。

如果需要获取`userFactoryBean`，那么beanName需要加上“&”，即`&userFactoryBean`。

### 两者对比

| 对比     | BeanFactory                                                  | FactoryBean<T>                                               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 类型     | 接口                                                         | 接口                                                         |
| 作用     | 生产bean的工厂，基础型IoC容器                                | 能生产或者修饰对象生成的工厂Bean，实现该接口可以定制实例化bean |
| 主要方法 | getBean(String name)<br/>getBean(Class<T> requiredType)<br/>isSingleton(String name) | getObject()<br/>getObjectType()<br/>isSingleton()            |

