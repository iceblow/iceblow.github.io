---
layout: post
title: Spring揭秘之IoC容器-BeanFactory
categories: [框架,Java基础]
description: Spring揭秘之IoC容器
keywords: spring, ioc,BeanFactory
---

Spring的IoC容器是一个IoC Service Provider，但这只是它被冠以IoC之名的部分原因，我们不能忽略的是“容器”。

Spring的IoC容器是一个提供IoC支持的轻量级容器，除了基本的IoC支持，还提供如AOP框架支持、企业级服务集成等服务。

Spring的IoC容器和IoC Service Provider所提供的服务之间存在一定的交集，二者的关系如图。

![image-20220113163459443](https://raw.githubusercontent.com/iceblow/images/main/20220325170050.png)

Spring提供了两种容器类型：`BeanFactory`和`ApplicationContext`。

### BeanFactory

**基础类型IoC容器**，提供完整的IoC服务支持。如果没有特殊指定，**默认采用延迟初始化策略**（lazy-load）。

只有当需要访问容器中的某个受管对象的时候，才进行初始化以及依赖注入操作。所以，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的IoC容器选择。

BeanFactory，顾名思义，就是生产Bean的工厂，可以完成作为IoC Service Provider的所有职责，包括**业务对象的注册和对象间依赖关系的绑定。**

以下是`BeanFactory`的源码：

```java
public interface BeanFactory {

	String FACTORY_BEAN_PREFIX = "&";

	Object getBean(String name) throws BeansException;

	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	boolean containsBean(String name);

	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	@Nullable
	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

	String[] getAliases(String name);
}
```

上面基本上都是查询相关的方法，例如，取得某个对象的方法（getBean）、查询某个对象是否存在于容器中的方法（containsBean），或者取得某个bean的状态或者类型的方法等。

因为通常情况下，对于独立的应用程序，只有主入口类才会跟容器的API直接耦合。有了BeanFactory，我们通常只需将“生产线图纸”交给BeanFactory，让BeanFactory为我们生产一个FXNewsProvider，如以下代码所示：

```java
BeanFactory container = new XmlBeanFactory(new ClassPathResource("配置文件路径")); 
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider"); 
newsProvider.getAndPersistNews();
```

#### 对象注册与依赖绑定方式

BeanFactory作为一个IoC Service Provider，为了能够明确管理各个业务对象以及业务对象之间的依赖绑定关系，同样需要某种途径来记录和管理这些信息。

##### 直接编码方式

```java
/**
 * BeanFactory 编码方式注入
 */
public class IocService4 {

    private AnimalService animalService;

    public IocService4() {
    }

    public IocService4(AnimalService animalService) {
        this.animalService = animalService;
    }

    public AnimalService getAnimalService() {
        return animalService;
    }

    public void setAnimalService(AnimalService animalService) {
        this.animalService = animalService;
    }

    // 测试
    public void test() {
        animalService.run();
    }
}
```

```java
public class Test {

    public static void main(String[] args) {

        DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
        BeanFactory container = (BeanFactory) bindViaCode(beanRegistry);
        IocService4 service4Provider = (IocService4) container.getBean("service4Provider");
        service4Provider.test();
    }

    public static BeanFactory bindViaCode(BeanDefinitionRegistry registry) {
        AbstractBeanDefinition service4Provider = new RootBeanDefinition(IocService4.class);
        AbstractBeanDefinition dogService = new RootBeanDefinition(DogService.class);

        // 将bean定义注册到容器中
        registry.registerBeanDefinition("service4Provider", service4Provider);
        registry.registerBeanDefinition("dogService", dogService);

        // 指定依赖关系
        //1. 可以通过构造方法注入方式（需要在service定义构造方法）
        ConstructorArgumentValues argValues = new ConstructorArgumentValues();
        argValues.addIndexedArgumentValue(0, dogService);
        service4Provider.setConstructorArgumentValues(argValues);

        // 2. 或者通过setter方法注入方式（需要先定义getter和setter方法）
        MutablePropertyValues propertyValues = new MutablePropertyValues();
        propertyValues.addPropertyValue(new PropertyValue("animalService", dogService));
        service4Provider.setPropertyValues(propertyValues);

        // 绑定完成
        return (BeanFactory) registry;
    }
}
```

BeanFactory只是一个接口，我们最终需要一个该接口的实现来进行实际的Bean的管理,DefaultListableBeanFactory就是这么一个比较通用的BeanFactory实现类。DefaultListableBeanFactory除了间接地实现了BeanFactory接口，还实现了BeanDefinitionRegistry接口，该接口才是在BeanFactory的实现中担当Bean注册管理的角色。

基本上，BeanFactory接口只定义如何访问容器内管理的Bean的方法，各个BeanFactory的具体实现类负责具体Bean的注册以及管理工作。

BeanDefinitionRegistry接口定义抽象了Bean的注册逻辑。通常情况下，具体的BeanFactory实现类会实现这个接口来管理Bean的注册。

![image-20220114160911666](https://raw.githubusercontent.com/iceblow/images/main/20220325170059.png)

##### 外部配置文件方式

Spring的IoC容器支持两种配置文件格式：Properties文件格式和XML文件格式。

下面只讲解XML文件格式的加载。XML配置格式是Spring支持最完整，功能最强大的表达方式。

```java
public static BeanFactory bindViaXMLFile(BeanDefinitionRegistry registry) {  
	XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry); 
 	reader.loadBeanDefinitions("classpath:../news-config.xml"); 
	return (BeanFactory)registry; 
 	// 或者直接
 	//return new XmlBeanFactory(new ClassPathResource("../service4.xml")); //定义bean的xml
}
```

XmlBeanDefinitionReader负责读取Spring指定格式的XML配置文件并解析，之后将解析后的文件内容映射到相应的BeanDefinition，并加载到相应的BeanDefinitionRegistry中（在这里是DefaultListableBeanFactory）。这时，整个BeanFactory就可以放给客户端使用了。

##### 注解方式

如果要通过注解标注的方式为Service注入所需要的依赖，可以使用@Autowired以 及@Component对相关类进行标记。

#### XML之旅

每个业务对象作为个体，在Spring的XML配置文件中是与<bean> 元素一一对应的。

```xml
<bean id="djNewsListener" class="..impl.DowJonesNewsListener"> 
</bean>
```

id是唯一标志，class属性指定其类型，还可以使用name属性来指定<bean>的别名 （alias）。

当然还有其他标签，构造函数<constructor-arg>，setter()方法<property>等，具体使用自行查询。

通常情况下，可以直接通过之前提到的所有元素，来显式地指定bean之间的依赖关系。这样，容器在初始化当前bean定义的时候，会根据这些元素所标记的依赖关系，首先实例化当前bean定义所依赖的其他bean定义。

除了可以通过配置明确指定bean之间的依赖关系，Spirng还提供了根据bean定义的某些特点将相互依赖的某些bean直接自动绑定的功能。通过<bean>的autowire属性，可以指定当前bean定义采用某种类型的自动绑定模式。这样，你就无需手工明确指定该bean定义相关的依赖关系。

Spring提供了5种自动绑定模式，即no、byName、byType、constructor和autodetect。

##### 自动绑定模式

- no

容器默认的自动绑定模式，不采用任何形式的自动绑定，完全依赖手工明确配置各个bean之间的依赖关系。

- byName

按照类中声明的实例变量的名称，与XML配置文件中声明的bean定义的beanName的值进行匹配，相匹配的bean定义将被自动绑定到当前实例变量上。

如果没有找到，则不做设置。但如果找到多个，容器会告诉你它解决不了“该选用哪一个”的问题，你只好自己查找原因，并自己修正该问题。

- byType

容器会根据当前bean定义类型，分析其相应的依赖对象类型，然后到容器所管理的所有bean定义中寻找与依赖对象类型相同的bean定义。

byType只能保证，在容器中只存在一个符合条件的依赖对象的时候才会发挥最大的作用，如果容器中存在多个相同类型的bean定义，那么请采用手动明确配置。

- constructor

byName和byType类型的自动绑定模式是针对property的自动绑定，而constructor类型则是针对构造方法参数的类型而进行的自动绑定，它同样是byType类型的绑定模式。

- autodetect

这种模式是byType和constructor模式的结合体，如果对象拥有默认无参数的构造方法，容器会优先考虑byType的自动绑定模式。否则，会使用constructor模式。

##### 自动绑定与手动明确绑定

自动绑定和手动明确绑定各有利弊。自动绑定的优点如下：

- 有效减少手动敲入配置信息的工作量
- 即使为当前对象增加了新的依赖关系，但只要容器中存在相应的依赖对象，就不需要更改任何配置信息。

自动绑定的缺点：

- 自动绑定不如明确依赖关系一目了然
- 据类型（byType）匹配进行的自动绑定，如果系统中增加了另一个相同类型的bean定义，那么整个系统就会崩溃；根据名字（byName）匹配进行的自动绑定，如果把原来系统中相同名称的bean定义类型给换掉，就会造成问题

##### 延迟初始化 lazy-init

与BeanFactory不同，ApplicationContext在容器启动的时候，就会马上对所有的“singleton的bean定义”进行实例化操作，优点是可以及时发现问题，但是会造成容器启动时间延长。

有时，我们不想某些bean定义在容器启动后就直接实例化，就可以使用lazy-init。

仅指定lazy-init-bean的lazy-init为true，并不意味着容器就一定会延迟初始化该bean的实例。如果某个非延迟初始化的bean定义依赖于lazy-init-bean，那么毫无疑问，按照依赖决计的顺序，容器还是会首先实例化lazy-init-bean，然后再实例化后者。

##### bean 的 scope 

BeanFactory除了拥有作为IoC Service Provider的职责，作为一个轻量级容器，它还有着其他一些职责，其中就包括对象的生命周期管理。

scope用来声明容器中的对象的限定场景或对象的存活时间，即容器在对象进入其相应的scope之前，生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁这些对象。

Spring容器最初提供了两种bean的scope类型：singleton和prototype，但发布2.0之后，又引入了另外三种scope类型，即request、session和global session类型。不过这3种只有在支持Web应用的ApplicationContext中使用。

需要注意的一点是，不要因为名字的原因而与GoF所提出的Singleton模式相混淆，二者的语意是不同的：标记为singleton的bean是由容器来保证这种类型的bean在同一个容器中只存在一个共享实例；而Singleton模式则是保证在同一个Classloader中只存在一个这种类型的实例。

###### singleton

标记为拥有singleton scope的对象定义，在Spring的**IoC容器中只存在一个实例**，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为**第一次被请求而初始化之后，将一直存活到容器退出**，它与IoC容器“几乎”拥有相同的“寿命”。

######  prototype 

容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。但是只要准备完毕，并且对象**实例返回给请求方之后，容器就不再拥有当前返回对象的引用**，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。

所以，对于那些请求方不能共享使用的对象类型，应该将其bean定义的scope设置为prototype。

###### request

```xml
<bean id="requestProcessor" class="...RequestProcessor" scope="request"/> 
```

Spring容器，即XmlWebApplicationContext会为每个HTTP请求创建一个全新的Request Processor对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。当同时有10个HTTP请求进来的时候，容器会分别针对这10个请求返回10个全新的RequestProcessor对象实例，且它们之间互不干扰。从不是很严格的意义上说，request可以看作prototype的一种特例。

###### session

对于Web应用来说，放到session中的最普遍的信息就是用户的登录信息。

Spring容器会为每个独立的session创建属于它们自己的全新的对象实例。与request相比，除了拥有session scope的bean的实例具有比request scope的bean可能更长的存活时间，其他方面没什么差别。

###### global session

global session只有基于portlet的Web应用程序中才有意义，如果在普通的基于servlet的Web应用中使用，容器会将其作为普通的session类型的scope对待。

##### 工厂方法与FactoryBean 

**FactoryBean是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口**，请不要将其与容器名BeanFactory相混淆。FactoryBean，其主语是Bean，定语为Factory，也就是说，它本身与其他注册到容器的对象一样，只是一个Bean而已，只不过，这种类型的Bean本身就是生产对象的工厂。

当某些对象的实例化过程过于烦琐，或者某些第三方库不能直接注册到Spring容器的时候，就可以实现org.springframework.beans.factory.FactoryBean接口，给出自己的对象实例化逻辑代码。当然，不使用Fac

toryBean，而像通常那样实现自定义的工厂方法类也是可以的。

FactoryBean接口只定义了三个方法，如下：

```java
public interface FactoryBean<T> {
  String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";
  
  //返回生产的对象实例
  @Nullable
	T getObject() throws Exception;
  
  // 对象的类型
  @Nullable
	Class<?> getObjectType();
  
  //对象是否单例，默认是
  default boolean isSingleton() {
		return true;
	}
}
```

### ApplicationContext

ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等。

ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说，会要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。

在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型容器是比较合适的。

详细内容会在后面的文章中介绍。

