---
layout: post
title: 【多线程】聊下ThreadLocal
categories: [多线程]
description: ThreadLocal
keywords: ThreadLocal, 多线程
---

接触并发编程的都知道ThreadLocal，简单来说，这个类能使线程中的某个值与保存值的对象关联起来，它提供了get与set等访问接口或方法，这些方法为每个使用该变量的线程都存有一个独立的副本，因此get总是返回由当前执行线程在调用set时设置的最新值。

### ThreadLocal 是什么

我们先看JDK的源码，官方原文部分描述如下：

> ```
> * This class provides thread-local variables.  These variables differ from
> * their normal counterparts in that each thread that accesses one (via its
> * {@code get} or {@code set} method) has its own, independently initialized
> * copy of the variable.  {@code ThreadLocal} instances are typically private
> * static fields in classes that wish to associate state with a thread (e.g.,
> * a user ID or Transaction ID).
> 
>  <p>Each thread holds an implicit reference to its copy of a thread-local
>  * variable as long as the thread is alive and the {@code ThreadLocal}
>  * instance is accessible; after a thread goes away, all of its copies of
>  * thread-local instances are subject to garbage collection (unless other
>  * references to these copies exist).
> ```

我们翻译过来就是：

> 该类提供线程局部变量。这些变量不同的是，每个线程有自己的独立的初始化变量的副本。
>
> ThreadLocal在类中的实例通常是静态私有的属性，用来关联线程的某些状态（例如，用户id或者事务id）。
>
> 每个线程都持有对其本地线程副本的隐式引用，只要线程还活着并且 ThreadLocal实例是可访问的； 
>
> 在一个线程消失后，它的所有副本线程本地实例会被垃圾回收（除非存在对这些副本的引用）。

### ThreadLocal 源码分析

```java
public class ThreadLocal<T> {
	
  private final int threadLocalHashCode = nextHashCode();
  
  private static AtomicInteger nextHashCode =
        new AtomicInteger();
  
  private static final int HASH_INCREMENT = 0x61c88647;
  
  private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
  }
  
  public ThreadLocal() {
  }
  
  //此处省略get、set、remove
  ......
}

```

我们重点分析 set(value)、get() 和 remove()方法。

#### set()方法

```java
public void set(T value) {
  Thread t = Thread.currentThread();
  ThreadLocalMap map = getMap(t);
  if (map != null)
    map.set(this, value);
  else
    createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
  return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
  t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

ThreadLocalMap 是ThreadLocal的内部静态类，我们暂不分析它。

ThreadLocal的set()方法逻辑：

- 首先，获取当前的线程
- 获取当前线程的 ThreadLocalMap
- 如果map不为null，设置**map的key为当前ThreadLocal的实例，value为要设置的值，也就是（threadLocal, value）**
- 如果map为null，说明是首次设置，需要创建一个ThreadLocalMap，通过构造方法赋值。

####   get()方法

```java
public T get() {
  Thread t = Thread.currentThread();
  ThreadLocalMap map = getMap(t);
  if (map != null) {
    ThreadLocalMap.Entry e = map.getEntry(this);
    if (e != null) {
      @SuppressWarnings("unchecked")
      T result = (T)e.value;
      return result;
    }
  }
  return setInitialValue();
}

 T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
      map.set(this, value);
    else
      createMap(t, value);
    return value;
}
```

ThreadLocal的get()方法逻辑：

- 获取当前线程，得到当前线程的ThreadLocalMap
- 如果map不为null，返回map的value
- 如果map为null，需要初始化ThreadLocalMap，并设置当前value为null，并返回。

#### remove()方法

```java
 public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
       m.remove(this);
 }
```

ThreadLocal的get()方法逻辑：

- 获取当前线程的ThreadLocalMap
- 如果map不为null，把map中的当前ThreadLocal的实例移除。

#### 静态内部类ThreadLocalMap

我们来看下JDK对ThreadLocalMap的描述。

> ```
> * ThreadLocalMap is a customized hash map suitable only for
> * maintaining thread local values. No operations are exported
> * outside of the ThreadLocal class. The class is package private to
> * allow declaration of fields in class Thread.  To help deal with
> * very large and long-lived usages, the hash table entries use
> * WeakReferences for keys. However, since reference queues are not
> * used, stale entries are guaranteed to be removed only when
> * the table starts running out of space.
> ```

翻译过来如下：

> ThreadLocalMap 是一个自定义的哈希映射，仅适用于维护线程本地值。
>
>  该类是包私有的， 允许在 Thread类 中声明字段。
>
>  用来帮助处理非常大且长期使用的对象，哈希表entries使用key的弱引用。
>
>  然而，由于引用队列不在使用时，老的的entries 保证仅在表开始空间不足情况下被删除。

```java
static class ThreadLocalMap {

 	// Entry 继承弱引用 类
  static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
      super(k);
      value = v;
    }
  }

  private static final int INITIAL_CAPACITY = 16;

  private Entry[] table;
  
  // 构造方法
  ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
  }
  
  ......
}
```

ThreadLoca类的 getMap方法是获取当前线程的ThreadLocalMap。

```java
 ThreadLocalMap getMap(Thread t) {
   // Thread类的 threadLocals 属性
   return t.threadLocals;
 }
```

```java
// Thread类属性
// 维护与该线程有关的ThreadLocal的值 
ThreadLocal.ThreadLocalMap threadLocals = null;
```

### 如何使用

我们比较常用的就是 在应用上下文中获取用户登录认证后的会话信息，比如username。

先定义静态私有ThreadLocal变量：

```java
public class SessionLocal {
		// username 线程副本变量
    private static ThreadLocal<String> userNameLocal = new ThreadLocal<>();
  
  	public static String getUserName() {
        return userNameLocal.get();
    }

    public static void setUserName(String userName) {
        userNameLocal.set(userName);
    }
  
      public static void removeUserName() {
        userNameLocal.remove();
    }
}
```

设置ThreadLocal的value，比如我们在拦截器的preHandle()里设置value

```java
@Override
public boolean preHandle(HttpServletRequest httpServletRequest,
                         HttpServletResponse httpServletResponse,
                         Object object) {
		// 获取登录信息...
  	String username = "用户名";
  	SessionLocal.set(username);
}
```

在单线程下，我们在API接口的应用上下文中，任一方法处，均可以获取当前用户的username，避免了我们手动传递该参数。

```java
String username = SessionLocal.getUserName();
```

完成调用后，移除ThreadLocal变量，例如，在拦截器的afterCompletion()方法移除ThreadLocal变量。

```java
@Override
public void afterCompletion(HttpServletRequest request,
                            HttpServletResponse response,
                            Object o, Exception e){
	SessionLocal.removeRealName();
}
```

### 使用场景

- 在进行对象跨层传输时，使用ThreadLocal可以避免多次传递，打破层次间的约束性
- 线程间数据隔离
- 进行事务操作，用于存储线程事务信息。
- 数据库连接，Session会话管理。

### 存在的问题

如果我们使用完ThreadLocal后，并没有调用remove()方法，那么ThreadLocalMap中还继续存在该ThreadLocal的key，如果线程继续存活（例如netty框架使用了线程池，或者我们业务自定义了线程池，线程一直存活），那么ThreadLocalMap就不会被回收，造成内存泄露。

### 最后

我们重点掌握ThreadLocal的使用场景，使用完后，一定要调用remove()方法清除线程副本变量，避免内存泄露的风险。
