---
layout: post
title: ArrayList和LinkList的源码分析
categories: JAVA基础
description: 
keywords: ArrayList,LinkList

---

## ArrayList和LinkList的源码分析 ##

### 概要 
ArrayList和LinkList是常用的存储结构,不看源码先分析字面意思,Array意思是数组,可知其底层是用数组实现的,Link意思是链接,可知是以链表实现,这两种数据结构各有什么特点呢?在实际开发中,我们要如何选择?
### 1.ArrayList    ###

- ArrayList是实现了List接口的可变数组,即动态数组,它不仅实现了List的可选操作,同时允许元素为null,它提供了一些方法来操作数组的大小来保存元素,该类大致相当于Vector,只是ArrayList不是线程安全的. 
-  每个ArrayList实例都有一个容量capacity,它是存储元素的数组的大小,其值至少等于list的size,当元素添加到ArrayList中,它的capacity会自动增大.
-  注意ArrayList的实现是非线程安全的,如果多个线程同时访问ArrayList实例,且至少有一个线程修改了元素,那么必须要加锁synchronized.
- 下面是ArrayList的类关系图,可以看出其继承了AbstractList,实现了List, RandomAccess, Cloneable, java.io.Serializable接口.  



![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/CA1B327F53B54F688852D5DF35572B0B/2184)

#### 1.1基本属性 ####
	public class ArrayList<E> extends AbstractList<E>
	    implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
	    private static final long serialVersionUID = 8683452581122892189L;
	
	    /**
	     * 默认初始容量10
	     */
	    private static final int DEFAULT_CAPACITY = 10;
	
	    /**
	     * 空实例的空数组
	     */
	    private static final Object[] EMPTY_ELEMENTDATA = {};
	
	    /**
	     * 默认空实例的空数组
	     */
	    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	
	    /**
	     * 存储元素的数组,它的大小就是ArrayList的容量,不用序列化
	     */
	    transient Object[] elementData; // non-private to simplify nested class access
	
	    /**
	     * ArrayList元素的大小
	     */
	    private int size;
	}


#### 1.2构造器 ####

	/**
	 * 构造空的list,并指定初始容量
	 *
	 * @param  initialCapacity  初始容量
	 * @throws IllegalArgumentException 如果初始容量为负数
	 */
	public ArrayList(int initialCapacity) {
	    if (initialCapacity > 0) {
	        this.elementData = new Object[initialCapacity];
	    } else if (initialCapacity == 0) {
	        this.elementData = EMPTY_ELEMENTDATA;
	    } else {
	        throw new IllegalArgumentException("Illegal Capacity: "+
	                                           initialCapacity);
	    }
	}
	
	/**
	 * 构造空的list,并指定初始容量10
	 */
	public ArrayList() {
	    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
	}
	
	/**
	 * 构造list,包含指定集合的所有元素
	 *
	 * @param c 指定的集合
	 * @throws NullPointerException 如果指定集合为null
	 */
	public ArrayList(Collection<? extends E> c) {
	    elementData = c.toArray();
	    if ((size = elementData.length) != 0) {
	        // c.toArray might (incorrectly) not return Object[] (see 6260652)
	        if (elementData.getClass() != Object[].class)
	            elementData = Arrays.copyOf(elementData, size, Object[].class);
	    } else {
	        // replace with empty array.
	        this.elementData = EMPTY_ELEMENTDATA;
	    }
	}
ArrayList list = new ArrayList(),默认容量为10,如果添加超过10个元素,就会造成list的扩容,内部会创建一个新的数组,对效率会有影响,如果有大量的元素添加,那么list就会频繁扩容,效率低下.因此在开发中可以指定初始容量,细节之中可以看出一个人的基本功.



#### 1.3常用方法 ####
	/**
	 * 返回list元素的数量
	 */
	public int size() {
	    return size;
	}
	
	/**
	 * 判断list是否包含元素,返回true或false
	 */
	public boolean isEmpty() {
	    return size == 0;
	}
	
	/**
	 * 判断是够包含指定的元素,返回true或false
	 */
	public boolean contains(Object o) {
	    return indexOf(o) >= 0;
	}
	
	/**
	 * 返回元素第一次出现的索引index
	 * 如果list没有这个元素,返回-1,否则返回第一次出现的index
	 * 注意:也可以查询null的索引
	 */
	public int indexOf(Object o) {
	    if (o == null) {
	        for (int i = 0; i < size; i++)
	            if (elementData[i]==null)
	                return i;
	    } else {
	        for (int i = 0; i < size; i++)
	            if (o.equals(elementData[i]))
	                return i;
	    }
	    return -1;
	}
	
	/**
	 * 返回元素最后一次出现的索引index
	 * 如果list没有这个元素,返回-1,否则返回最后一次出现的index
	 * 注意:也可以查询null的索引
	 */
	public int lastIndexOf(Object o) {
	    if (o == null) {
	        for (int i = size-1; i >= 0; i--)
	            if (elementData[i]==null)
	                return i;
	    } else {
	        for (int i = size-1; i >= 0; i--)
	            if (o.equals(elementData[i]))
	                return i;
	    }
	    return -1;
	}
	
	/**
	 * 返回指定索引的元素
	 * @param  index 索引
	 * @return 元素
	 * @throws IndexOutOfBoundsException, 如果index超出数组的范围
	 */
	public E get(int index) {
	    rangeCheck(index);
	
	    return elementData(index);
	}
	
	/**
	 * 替换指定索引的元素
	 *
	 * @param index 索引
	 * @param element 新元素
	 * @return 老元素
	 * @throws IndexOutOfBoundsException 
	 */
	public E set(int index, E element) {
	    rangeCheck(index);
	
	    E oldValue = elementData(index);
	    elementData[index] = element;
	    return oldValue;
	}
下面重点介绍add()和remove()方法,元素的新增和删除内部是如何实现的呢?  

	public boolean add(E e) {
	    ensureCapacityInternal(size + 1);  // Increments modCount!!
	    elementData[size++] = e;
	    return true;
	}
ArrayList新增元素时,也就是在list的尾部添加一个元素,首先修改元素的数量+1
> ensureCapacityInternal(size+1),即list的size+1,元素数量加1,详细实现如下:
> >     private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
元素的数量+1后,判断minCapacity有没有超出当前的容量,如果超出了,就要进行扩容操作grow()  
>>  
	private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
其逻辑如下:**新容量 = 老容量 + 老容量右移1位(即除以2)**,也就是大约1.5倍的老容量,为什么说大约呢?因为如果老容量是偶数,那么新容量正好等于1.5倍老容量,如果老容量是奇数11,那么新容量是15.  
容量在大,也是有限制的,最大MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8,有21亿,估计没有人会放这么多的数据吧.   

>elementData[size++] = e, 然后把新元素e放在新索引的位置,也就是数组的尾部.   

ArrayList删除元素有两种,一种是根据索引删除,二是直接删除对象  

- 根据索引删除对象             
       
	    public E remove(int index) {
	        rangeCheck(index);
	        modCount++;
	        E oldValue = elementData(index);
	        int numMoved = size - index - 1;
	        if (numMoved > 0)
	        System.arraycopy(elementData, index+1, elementData, index,
	         numMoved);
	        elementData[--size] = null; // clear to let GC do its work
	        return oldValue;
	    }
  numMoved = 4 - 2- 1 = 1;  
  System.arraycopy(elementData, 3, elementData, 2,1);  
  流程图如下:   
  ![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/DC58FA64275842FEB5E82318804BAF4E/2191)

**本质上是把要删除的元素替换为它后面的元素,然后把最后一个元素赋值为null,手动GC,返回老的元素.**

- 直接删除对象  

		public boolean remove(Object o) {
		    if (o == null) {
		        for (int index = 0; index < size; index++)
		            if (elementData[index] == null) {
		                fastRemove(index);
		                return true;
		            }
		    } else {
		        for (int index = 0; index < size; index++)
		            if (o.equals(elementData[index])) {
		                fastRemove(index);
		                return true;
		            }
		    }
		    return false;
		}
就是循环遍历数组,当元素第一次出现的时候,删除这个元素,同时返回true,如果元素不存在,那么数组不改变,同时返回false.  
### 2.LinkList ###

- LinkList是List接口的双向链表实现,不仅实现了List的方法,同时允许元素为null
- 获取index索引的元素时,会从链表的头部或尾部进行查找,哪边近从哪边开始
- 注意该实现是线程非安全  
下面是LinkList的类关系图,它继承了AbstractSequentialList,实现了List, Deque, Cloneable, java.io.Serializable接口  
![](https://note.youdao.com/yws/public/resource/13e45ac31576fe901e85f3c64ac06332/xmlnote/406417019D3E49ADA952DF44F3E15CB1/2189)
#### 2.1基本属性 ####
	public class LinkedList<E> extends AbstractSequentialList<E>
	implements List<E>, Deque<E>, Cloneable, java.io.Serializable{
	    transient int size = 0;
	
	    /**
	     * 第一个节点,不用序列化
	     */
	    transient Node<E> first;
	
	    /**
	     * 最后一个节点,不用序列化
	     */
	    transient Node<E> last;
	}
#### 2.2构造器 ####
	/**
	 * 构造空的list
	 */
	public LinkedList() {
	}
	
	/**
	 * 构造指定集合的list
	 * @param  c 集合
	 * @throws NullPointerException 如果集合为null
	 */
	public LinkedList(Collection<? extends E> c) {
	    this();
	    addAll(c);
	}
#### 2.3常用方法
添加元素add()   

    /**
     * 在LinkList的尾部添加元素
     * 这个方法相当于addLast
     * 返回true/false
     */
    public boolean add(E e) {
    	linkLast(e);
     	return true;
    }

> linkLast的具体实现如下:
>>     void linkLast(E e) {
		//l赋值为last节点
	    final Node<E> l = last;
		//创建新的节点e,前面元素是l,后面是null
	    final Node<E> newNode = new Node<>(l, e, null);
		//把新的节点标记为last节点
	    last = newNode;
		//判断是不是第一个节点
		//如果是,新的节点就是第一个节点
	    if (l == null)
	    	first = newNode;
	    else
		//如果不是,之前的最后一个节点后面是新的节点
	    	l.next = newNode;
	    size++;
	    modCount++;
	  }

删除元素remove()  

 	public boolean remove(Object o) {
 	    if (o == null) {
 	        for (Node<E> x = first; x != null; x = x.next) {
 	            if (x.item == null) {
 	                unlink(x);
 	                return true;
 	            }
 	        }
 	    } else {
 	        for (Node<E> x = first; x != null; x = x.next) {
 	            if (o.equals(x.item)) {
 	                unlink(x);
 	                return true;
 	            }
 	        }
 	    }
 	    return false;
 	}

> 从头部遍历所有的节点,如果当前节点元素与要删除的节点元素相同,那么移除当前节点,如果LinkList包含多个要删除的元素,那么只会删除index较小的那个节点.
### 3.总结 ###
1. ArrayList实现了RandomAccess,随机读取数据速度较快,它有索引index,会直接读取数组的索引,效率很高,但是新增和删除元素,内部进行数组的动态复制,效率低，如果遇到大量的数据add操作，频繁扩容，性能更差
2. LinkList新增元素就是创建了一个新的节点,只要把last节点指向最后一个元素即可,效率很高,删除元素,就是移除指定的节点,同时调整前后的节点即可,但是对于随机访问元素,它需要判断当前index离头部和尾部哪个更近,然后去依次查找,效率低下
3. 所以对于随机快速读取数据,可以使用ArrayList,对于快速新增和删除元素,可以使用LinkList