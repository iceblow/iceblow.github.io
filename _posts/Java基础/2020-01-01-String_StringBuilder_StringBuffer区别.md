---
layout: post
title: String、StringBuilder、StringBuffer区别
categories: Java基础
description: 
keywords: 
---

String是Java中最常用的类之一，那么它是如何实现的呢，下面我们来分析下JDK源码。

### 1.String

以下代码是JDK1.8源码：

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {

	//存储字符,final修饰
    private final char value[];

    //缓存hash code,默认0
    private int hash;

    //序列号
    private static final long serialVersionUID = -6849794470754667710L;

    //声明可序列化字段
    private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
}
```
#### 1.1 基本属性

* char value[],用来存储字符串对象的字符数组
* int hash,用来缓存字符串的hash code,默认值为0
* long serialVersionUID,用来序列化的序列版本号
* ObjectStreamField[],可序列化类的字段说明

#### 1.2 常用构造器  

```java
public String() {
	this.value = "".value;
}
```
初始化新创建的对象,表示空字符串""。请注意,此构造函数是不需要使用的，因为字符串是不可变的.  
String str = new String(); 本质上是创建了一个空的字符数组,str的长度为0   

```java
public String(String original) {
	this.value = original.value;
	this.hash = original.hash;
}   
```

初始化新创建的对象,表示和参数一样的字符串,换句话说是创建了和参数一样的对象副本,除非需要显示的声明副本,否则该构造函数是不需要的,因为字符串是不可变的  

```java
public String(StringBuffer buffer) {
    synchronized(buffer) {
    this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
```

把StringBuffer的内容复制到String对象中,随后修改StringBuffer对象的值,并不会影响String对象  

```java
public String(StringBuilder builder) {
	this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```
把StringBuilder的内容复制到String对象中,随后修改StringBuilder的值,并不会影响String对象;  
此构造函数是为了把StringBuilder转移到String对象,但是推荐使用StringBuilder的toString()方法,因为运行更快

#### 1.3 常用方法

```java
//返回字符串的长度
public int length() {
	return value.length;
}
  
//比较两个String值是否相等
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}

//生成hash code
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

**equals方法的判断流程:**

* 首先判断两个对象是否相同,若相同返回true;若不同,下一步  
* 判断参数是否为String对象,若不是,返回false;若是,下一步  
* 判断两个String的长度是否相等,若不是,返回false;若是,下一步  
* 按字符数组索引依次比较字符,如果有任一不相同,返回false,否则返回true  

这里可以看到equals方法的实现也是调用了==来比较对象是否相同,因此在讨论equals和==区别的时候
要全面分析.

#### 1.4 为什么说String是不可变对象?   

存储字符的数组value[]是final修饰的,值不可更改.

### 2. AbstractStringBuilder

可变的字符序列,StringBuilder和StringBuffer都继承了该类,要了解StringBuilder和StringBuffer首先先了解AbstractStringBuilder.

#### 2.1 基本属性 ####
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
}
```
char[] value: 存储字符的数组  
int count: 存储的字符的数量

#### 2.2 构造器 ####
```java
/**
 * 无参构造器,用于子类序列化
 */
AbstractStringBuilder() {
}

/**
 * 指定字符数组容量
 */
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```
#### 2.3 常用方法 ####
```java
 	/**
     * 返回字符的数量
     */
    @Override
    public int length() {
        return count;
    }
```



```java
/**
 * 返回当前可存储字符的最大数量,即容量
 */
public int capacity() {
    return value.length;
}

/**
 * 保证当前容量大于等于指定的最小数量minimumCapacity,会调用扩容方法
 */
public void ensureCapacity(int minimumCapacity) {
    if (minimumCapacity > 0)
        ensureCapacityInternal(minimumCapacity);
}

/**
 * 扩容,只有minimumCapacity大于当前容量,才会copy数组扩容
 */
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
/**
 * 当前对象拼接字符串str
 * 如果参数为null,那么最终字符串为"null",如果参数类型为boolean,那么返回的是"true"或"false"
 * 例1: "abc".append(null) = "abcnull"
 * 例2: "abc".append(false) = "abcfalse"
 */
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```
### 3.StringBuilder

可变的字符序列,非线程安全,StringBuilder和StringBuffer的实现方法很相似,区别在于是否线程安全,在单线程的情况下可使用StringBuilder,因为它比StringBuffer运行更快.StringBuilder继承了AbstractStringBuilder类.

#### 3.1 基本属性 ####

继承父类

#### 3.2 构造器 ####

```java
/**
 * 默认容量16
 */
public StringBuilder() {
    super(16);
}

/**
 * 指定初始容量
 */
public StringBuilder(int capacity) {
    super(capacity);
}

/**
 * 把String字符串初始化到对象中,容量变为str的长度+16
 */
public StringBuilder(String str) {
    super(str.length() + 16);
    append(str);
}

/**
 * 把字符初始化到对象中,容量变为字符的长度+16
 */
public StringBuilder(CharSequence seq) {
    this(seq.length() + 16);
    append(seq);
}
```
#### 3.3 常用方法 ####
同父类

### 4. StringBuffer

可变的字符序列,线程安全,StringBuffer继承了AbstractStringBuilder类.

#### 4.1 基本属性 ####
继承父类,同时还有属性toStringCache  
这里涉及到一个关键字**transient**,其作用是让某些被修饰的成员属性变量不被序列化,为什么不需要序列化呢?主要是因为这个变量可能为null,也可能非空,不能准确的代表StringBuffer的属性.所以没有必要序列化,也节省了空间.

```java
public final class StringBuffer extends AbstractStringBuilder
	implements java.io.Serializable, CharSequence{

    /**
     * 最后一次调用toString返回值的缓存
     * 当StringBuffer被修改时该缓存被清除
     */
    private transient char[] toStringCache;

    /** 序列号 */
    static final long serialVersionUID = 3388685877147921107L;
}
```
#### 4.2 构造器 ####
同StringBuilder

#### 4.3 常用方法 ####
与StringBuilder方法基本相同,区别在于在StringBuilder的方法上加了synchronized锁.
不同的地方还包括每个修改对象的方法,比如append方法都会清除toStringCache缓存.

```java
@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
@Override
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}
```
1. 在调用toString方法的时候,会先判断缓存toStringCache是否存在,如果不存在,那么把当前对象赋值给toStringCache,然后得到toStringCache的字符串;如果toStringCache已经存在,那么直接读取缓存的字符串.  
2. toStringCache是否存在依赖于StringBuffer对象是否被修改,如果被修改了,那么缓存被清空.

### 5.总结 ###
1. String,StringBuilder,StringBuffer这三个类都可以创建和操作字符串;  
2. 不同点在于String是不可变的,不存在线程安全问题;StringBuilder和StringBuffer字符串是可变的,StringBuilder线程不安全,StringBuffer线程安全;  
3. 对于简单的字符串赋值和拼接操作,可使用String  
4. 对于字符串频繁修改和拼接操作,不建议使用String,因为每次操作都需要创建一个String对象,单线程推荐使用StringBuilder,多线程推荐使用StringBuffer.
