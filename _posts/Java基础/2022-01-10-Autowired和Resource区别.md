---
layout: post
title: Autowired和Resource区别
categories: [Java基础]
description: Autowired和Resource区别
keywords: Autowired,Resource
---

Autowired和Resource这两个注解是面试中经常问到的，bean自动装配用到的，他们两个有什么区别呢？

### @Resource

#### 装配顺序

默认按照byName方式进行装配，属于J2EE自带注解，没有指定name时，name指的是变量名

-  如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
- 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
- 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
- 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配

#### 源码

```java
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
public @interface Resource {
    /**
     * The JNDI name of the resource.  For field annotations,
     * the default is the field name.  For method annotations,
     * the default is the JavaBeans property name corresponding
     * to the method.  For class annotations, there is no default
     * and this must be specified.
     */
    String name() default "";

    /**
     * The Java type of the resource.  For field annotations,
     * the default is the type of the field.  For method annotations,
     * the default is the type of the JavaBeans property.
     * For class annotations, there is no default and this must be
     * specified.
     */
    Class<?> type() default java.lang.Object.class;
 
  	......
}
```



### @Autowired

#### 装配顺序

默认按byType自动注入，是Spring的注解

- 默认按类型装配（属于spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false；

- 按类型装配的过程中，如果发现找到多个bean，则又按照byName方式进行比对，如果还有多个，则报出异常；

#### 源码

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;

}
```



### Demo分析

#### 定义接口

```java
package com.example.demo.test;

public interface AnimalService {

    void run();
}

```

#### 实现类1

```java
package com.example.demo.test;

import org.springframework.stereotype.Component;

@Component
public class DogService implements AnimalService {

    @Override
    public void run() {
        System.out.println("dog run ");
    }
}

```

#### 实现类2

```java
package com.example.demo.test;

import org.springframework.stereotype.Component;

@Component
public class CatService implements AnimalService {

    @Override
    public void run() {
        System.out.println("cat run ");
    }
}

```

#### 测试

##### @Autowired装配

```
  @Autowired
  private AnimalService animalService;
```

![image-20220110135054344](https://cdn.jsdelivr.net/gh/iceblow/images/20220110135054.png)

IDEA会提示不能自动装配，因为AnimalService的bean type有两个。

我们可以在注入时加上`@Qualifier`注解指定bean name。

```java
@Autowired
@Qualifier("dogService") //默认是小写，如果在DogService自定义name，此处就是自定义name
private AnimalService animalService;

public void test() {
        animalService.run();
}
```

测试结果：dog run

如果注入相同的2个dogService呢？

```java
@Autowired
@Qualifier("dogService") // bean1
private AnimalService animalService;

@Autowired
@Qualifier("dogService") // bean2， 这两个是否相同？
private AnimalService animalService2;

public void test() {
  // bean是否相同
  System.err.println(animalService == animalService2);
  animalService2.run();
}
```

测试结果：true，两个bean是相同的。

##### @Resource装配

###### 默认方式，没有name和type

```java
@Resource
private AnimalService animalService2;
```

编译是没问题，但是服务启动后会报错：

```yaml
NoUniqueBeanDefinitionException: No qualifying bean of type 'com.example.demo.test.AnimalService' available: 
expected single matching bean but found 2: catService,dogService
```

![image-20220110143210825](https://cdn.jsdelivr.net/gh/iceblow/images/20220110143210.png)

###### 同时指定name和type

```java
// 同时指定name和type，结果ok
@Resource(name = "catService", type = CatService.class)
private AnimalService animalService2;
```

如果name和type不匹配呢？

```java
// name是catService， 而type是DogService，不匹配
@Resource(name = "catService", type = DogService.class)
private AnimalService animalService2;
```

服务启动报错：

```java
BeanNotOfRequiredTypeException: 
Bean named 'catService' is expected to be of type 'com.example.demo.test.DogService' but was actually of type 'com.example.demo.test.CatService'
```

###### 指定name

```
@Resource(name = "catService")
private AnimalService animalService2;
```

测试结果正常

###### 指定type

```
@Resource(type = CatService.class)
private AnimalService animalService2;
```

测试结果正常

### 小结

- 如果service只有一个实现类的时候，@Resource和@Autowired结果是一样的；
- @Autowired默认byType注入，如果找到多个bean，则按byName匹配，如果还是多个那么报错；可以和@Qualifier注解同时使用达到byName装配；
- @Resource默认byName装配，可以指定name和type装配，比较灵活，推荐使用。



