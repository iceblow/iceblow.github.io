---
layout: post
title: Spring揭秘之IoC容器-概念
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

相对于前两种注入方式来说，接口注入没有那么简单明了。

举个例子，被注入对象`DogService`如果想要`IocService`为其注入`DogService`依赖的对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。

```java
// 接口
public interface AnimalServiceCallable {

    // 注入依赖
    void injectAnimalService(AnimalService animalService);
}
```

```java
/**
 * 接口注入，需要实现注入接口
 */
public class IocService3 implements AnimalServiceCallable {

    private AnimalService animalService;

    @Override
    public void injectAnimalService(AnimalService animalService) {
        this.animalService = animalService;
    }

    public void test() {
        animalService.run();
    }
    
    public static void main(String[] args) {
        IocService3 service = new IocService3();
      	// 注入
        service.injectAnimalService(new DogService());
        service.test();
    }
}
```

相对于前两种依赖注入方式，接口注入比较死板和烦琐。如果需要注入依赖对象，被注入对象就必须声明和实现另外的接口。

#### 注入方式比较

- 接口注入：从注入方式的使用上来说，接口注入是现在不甚提倡的一种方式，因为它强制被注入对象实现不必要的接口，带有侵入性。

- 构造方法注入：优点就是，对象在构造完成之后，即已进入就绪状态，可以马上使用。缺点就是，当依赖对象比较多的时候，构造方法的参数列表会比较长。而通过反射构造对象的时候，对相同类型的参数的处理会比较困难，维护和使用上也比较麻烦。而且在Java中，构造方法无法被继承，无法设置默认值。对于非必须的依赖处理，可能需要引入多个构造方法，而参数数量的变动可能造成维护上的不便。

- setter方法注入：因为方法可以命名，所以setter方法注入在描述性上要比构造方法注入好一些。另外，setter方法可以被继承，允许设置默认值，而且有良好的IDE支持。缺点当然就是对象无法在构造完成后马上进入就绪状态。

综上所述，构造方法注入和setter方法注入因为其侵入性较弱，且易于理解和使用，是使用最多的注入方式。

### IoC的优点

要说IoC模式能带给我们什么好处，首先能想到的就是以下几条，

- 不会对业务对象构成很强的侵入性
- 对象具有更好的可测试性、可重用性和可扩展性等。

当然优点不止这些，总之，IoC是一种可以帮助我们解耦各业务对象间依赖关系的对象绑定方式。

### IoC的职责

IoC Service Provider的职责相对来说比较简单，主要有两个：业务对象的构建管理和业务对象间

的依赖绑定。

- **业务对象的构建管理**

  在IoC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但这部分工作需要IoC Service Provider将对象的构建逻辑从客户端对象那里剥离出来，以免这部分逻辑污染业务对象的实现

- **业务对象间的依赖绑定**

  对于IoC Service Provider来说，这个职责是最艰巨也是最重要的，这是它的最终使命之所在。。IoC Service Provider通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。

#### 如何管理对象的依赖关系

##### 直接编码方式

```java
IoContainer container = ...; 
container.register(FXNewsProvider.class,new FXNewsProvider()); 
container.register(IFXNewsListener.class,new DowJonesNewsListener()); 
... 
FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class); 
newProvider.getAndPersistNews();
```

通过为相应的类指定对应的具体实例，可以告知IoC容器，当我们要这种类型的对象实例时，请将容器中注册的、对应的那个具体实例返回给我们。

##### 配置文件方式

这是一种较为普遍的依赖注入关系管理方式。像普通文本文件、properties文件、XML文件等，都可以成为管理依赖注入关系的载体。

不过，最为常见的，还是通过XML文件来管理对象注册和对象间依赖关系。

```xml
<bean id="newsProvider" class="..FXNewsProvider"> 
  //属性1
 <property name="newsListener"> 
 <ref bean="djNewsListener"/> 
 </property> 
  //属性2
 <property name="newPersistener"> 
 <ref bean="djNewsPersister"/> 
 </property> 
</bean> 
<bean id="djNewsListener" 
 class="..impl.DowJonesNewsListener"> 
</bean> 
<bean id="djNewsPersister" 
 class="..impl.DowJonesNewsPersister"> 
</bean>
```

通过“newsProvider”这个名字，从容器中取得已经组装好的FXNewsProvider并直接使用。

```java
container.readConfigurationFiles(...); 
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("newsProvider"); 
newsProvider.getAndPersistNews();
```

##### 元数据方式

这种方式的代表实现是Google Guice，我们可以直接在类中使用元数据信息来标注各个对象之间的依赖关系，然后由Guice框架根据这些注解所提供的信息将这些对象组装后，交给客户端对象使用。

此处代码省略。

### 小结

本章主要介绍了IoC的概念，讨论了3种基本的依赖注入方式，以及使用IoC模式的职责。



