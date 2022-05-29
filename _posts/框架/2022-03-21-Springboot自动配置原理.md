---
layout: post
title: Springboot自动配置原理
categories: [框架]
description: some word here
keywords: keyword1, keyword2
---

Springboot自动配置原理也算是面试中的热点题目，常用框架的原理还是必须掌握的。下面我们通过源码分析其原理。

首先，Springboot项目启动类引入注解`@SpringBootApplication`：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
                                 @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
  
  // 排除自动配置类
  @AliasFor(annotation = EnableAutoConfiguration.class)
  Class<?>[] exclude() default {};

  @AliasFor(annotation = EnableAutoConfiguration.class)
  String[] excludeName() default {};
}
```

该注解会引用`@EnableAutoConfiguration`，间接引入`AutoConfigurationImportSelector`类。

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
	Class<?>[] exclude() default {};
	String[] excludeName() default {};

}
```

下面我们重点介绍`AutoConfigurationImportSelector`类，该类实现了`selectImports`方法，

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return NO_IMPORTS;
  }
  AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
  return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```



```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
  if (!isEnabled(annotationMetadata)) {
    return EMPTY_ENTRY;
  }
  // exclude和excludeName
  AnnotationAttributes attributes = getAttributes(annotationMetadata);

  // 加载 META-INF/spring.factories 下的自动配置类
  List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);

  // 去重
  configurations = removeDuplicates(configurations);

  // 排除exclude类
  Set<String> exclusions = getExclusions(annotationMetadata, attributes);
  checkExcludedClasses(configurations, exclusions);
  configurations.removeAll(exclusions);

  // filter，默认OnClassCondition、OnWebApplicationCondition、OnBeanCondition条件注解
  configurations = getConfigurationClassFilter().filter(configurations);

  // 通知自动配置导入事件
  fireAutoConfigurationImportEvents(configurations, exclusions);

  // 封装返回配置类
  return new AutoConfigurationEntry(configurations, exclusions);
}
```

configurations截图如下：

![image-20220321120907243](https://img-1257951221.cos.ap-shanghai.myqcloud.com//20220321120907.png)

实现原理大致流程图如下：

![image-20220321175401454](https://img-1257951221.cos.ap-shanghai.myqcloud.com//20220321175401.png)
