---
layout: post
title: Spring揭秘之IoC容器
categories: [框架,Java基础]
description: Spring揭秘之IoC容器
keywords: spring, ioc
---

IoC作为轻量级容器，全称为Inversion of Control，中文翻译为“控制反转”。

好莱坞原则“Don’t call us, we will call you.” 恰如其分地表达了“反转”的意味。

那么，为什么需要IoC？IoC的具体意义是什么？它到底有什么独到之处？

下面我们举个例子来说明。

### 背景

如果我们依赖于某个类或服务，最简单而有效的方式就是直接在类的构造函数中新建相应的依赖类。

```java
public class IocService {

    private DogService dogService;
    private CatService catService;

  	// 如果DogService还有多个依赖对象呢（或有多层依赖）？
    public IocService(DogService dogService, CatService catService) {
        this.dogService = dogService;
        this.catService = catService;
    }

    public void testIoc() {
        dogService.run();
        catService.run();
    }
}
```

我们自己每次用到什么依赖对象DogService和CatService都要主动地去获取，这是否真的必要？

上面的例子我们依赖DogService，可以直接new一个对象，用起来感觉挺简单的，**如果依赖的对象还需要依赖其他对象，层层嵌套，那么主动去获取的代价就会很大**。

我们最终所要做的，其实就是直接调用依赖对象所提供的某项服务而已。只要用到这个依赖对象的时候，它能够准备就绪，我们完全可以不管这个对象是自己找来的还是别人送过来的。

实际上，IoC就是为了帮助我们避免之前的“大费周折”，而提供了更加轻松简洁的方式。它的反转，就反转在让你从原来的事必躬亲，转变为现在的享受服务。

![image-20220112184823366](https://gitee.com/dxyin/pic/raw/master/20220112184823.png)

### 概念

IoC模式最权威的总结和解释，控制反转和依赖注入，其中有三种依赖注入的方式，

- 构造方法注入（constructor injection）
- setter方法注入（setter injection）
- 接口注入（interface injection）

下面让我们详细看一下这三种方式的特点及其相互之间的差别。

#### 构造方法注入

构造方法注入，就是被注入对象可以通过在其构造方法中声明依赖对象的参数列表， 让外部（通常是IoC容器）知道它需要哪些依赖对象。

```java
// 构造方法注入
public IocService(DogService dogService, CatService catService) {
        this.dogService = dogService;
        this.catService = catService;
    }
```

构造方法注入方式比较直观，对象被构造完成后，即进入就绪状态，可以马上使用。

#### setter 方法注入

对于JavaBean对象来说，通常会通过setXXX()和getXXX()方法来访问对应属性。所以，当前对象只要为其依赖对象所对应的属性添加setter方法，就可以通过setter方法将相应的依赖对象设置到被注入对象中。

```java
/**
 * setter方法注入
 */
public class IocService2 {

    private DogService dogService;

    public DogService getDogService() {
        return dogService;
    }

 	 // setter
    public void setDogService(DogService dogService) {
        this.dogService = dogService;
    }

    public void testIoc() {
        dogService.run();
    }

    public static void main(String[] args) {
            IocService2 service2 = new IocService2();
            service2.setDogService(new DogService());
            service2.testIoc();
    }
}

```

setter方法注入虽不像构造方法注入那样，让对象构造完成后即可使用，但相对来说更宽松一些，可以在对象构造完成后再注入。

#### 接口注入



