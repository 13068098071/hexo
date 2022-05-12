---
title: LinkedList
date: 2021-12-24
description: LinkedList集合笔记
cover: https://s2.loli.net/2021/12/23/TNY612AUy38Dfk7.jpg
tags: List
categories: 集合
---
## LinkedList

### 概述

#### 基础知识

>什么是链表

链表是由一系列非连续的节点组成的存储结构，简单分下类的话，链表又分为单向链表和双向链表，而单向/双向链表又可以分为循环链表和非循环链表

>单向链表

单向链表就是通过每个结点的指针指向下一个结点从而链接起来的结构，最后一个节点的next指向null。

![单向列表](https://raw.githubusercontent.com/13068098071/picode/main/img/193331_JljJ_2927759.png)

>单向循环链表

单向循环链表和单向列表的不同是，最后一个节点的next不是指向null，而是指向head节点，形成一个“环”。

![单向循环链表](https://raw.githubusercontent.com/13068098071/picode/main/img/193412_xGR9_2927759.png)

>双向链表

双向链表是包含两个指针的，pre指向前一个节点，next指向后一个节点，但是第一个节点head的pre指向null，最后一个节点的tail指向null。

![双向链表](https://raw.githubusercontent.com/13068098071/picode/main/img/193440_9dt2_2927759.png)

>双向循环链表

双向循环链表和双向链表的不同在于，第一个节点的pre指向最后一个节点，最后一个节点的next指向第一个节点，也形成一个“环”。**而LinkedList就是基于双向循环链表设计的。**

![双向循环链表](https://raw.githubusercontent.com/13068098071/picode/main/img/193526_9m6M_2927759.png)



LinkedList底层是基于双向链表，链表在内存中不是连续的，而是通过引用来关联所有的元素，所以链表的优点在于添加和删除元素比较快，因为只是移动指针，并且不需要判断是否需要扩容，缺点是查询和遍历效率比较低。

#### 类图

![类图](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211224141813601.png) LinkedList底层是基于双向链表，链表在内存中不是连续的，而是通过引用来关联所有的元素，所以链表的优点在于添加和删除元素比较快，因为只是移动指针，并且不需要判断是否需要扩容，缺点是查询和遍历效率比较低。

- LinkedList是基于双向循环链表实现的，除了可以当做链表来操作外，它还可以当做栈、队列和双端队列来使用
- 实现了所有可选的List操作并且允许存储任何元素，包括Null
- LinkedList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了Cloneable接口，能被克隆
- LinkedList是非线程安全的，只在单线程下适合使用

### 源码分析

#### 成员变量的构造方法

```java
/**
 * 当前存储的元素个数
 */
transient int size = 0;
 
/**
 * 
 * 首节点
 */
transient Node<E> first;
 
/**
 * 末节点
 */
transient Node<E> last;
 
/**
 * 空构造器
 */
public LinkedList() {
}
 
/**
 *传入集合参数的构造器
 */
public LinkedList(Collection<? extends E> c) {
    this();//调用当前类的构造函数
    addAll(c);
}
```

####  添加方法

>add

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
void linkLast(E e) {
	/**
	 * 获取当前链表的最后一个节点
	 */
    final Node<E> l = last;
    /**
     * 创建一个以当前最后一个节点为之前节点的节点
     */
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    /**
     * 空表，首次插入
     */
    if (l == null)
        first = newNode;
    else
        l.next = newNode;//不是首次插入，则最后一个节点的后置节点地址赋值给新节点
    size++;
    modCount++;
}
```

>addAll

```java
/**
 *在链表的尾端追加指定集合的所有元素，按指定的迭代器的集合顺序返回，在这个操作执行总是如果指定的集合被修改了
 *，那么该行为操作将提示未定义
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
public boolean addAll(int index, Collection<? extends E> c) {
	/**
	 * 检查index是否越界，index=size+1
	 */
    checkPositionIndex(index);
    /**
     * 将集合参数转化为数组
     */
    Object[] a = c.toArray();
    int numNew = a.length;//要插入的集合长度
    if (numNew == 0)
        return false;
    /**
     * 定义pred和succ两个Node对象，用于标识要插入元素的前置节点和后置节点
     */
    Node<E> pred, succ;
    /**
     * 这里为什么要写if..else？
     * 因为该方法不一定是从上层方法addAll(size, c)过来的，还有可能是直接调用了addAll(int index, Collection<? extends E> c)
     * 方法，从上层addAll(size, c)跳转过来的，size=index也就从尾部插入，但是直接调用的该方法，则从传进来的参数index这个位置（肯能是任何位置）插入
     */
    if (index == size) {//表明是从尾部插入
        succ = null;//从尾部插入，后置节点为null
        pred = last;//从尾部插入，前置节点为当前LinkedList中的最后一个节点
    } else {//表明不是从尾部插入
        succ = node(index);//查到当前LinkedList中位置为index的节点并把它赋给要插入元素的后置节点
        pred = succ.prev;//把上一步得到的节点的前置节点赋值给要插入元素的后置节点
    }
 
    for (Object o : a) {//变量集合参数
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)//说明插入之前当前链表是空链表
            first = newNode;//新节点是第一个节点
        else
            pred.next = newNode;//设置插入元素的的前置节点的后置节点为新节点
        pred = newNode;//更改指向后将新节点对象赋给pred作为下次循环中新插入节点的前一个对象节点，依次循环
    }
  //此时pred代表集合元素的插入完后的最后一个节点对象
    if (succ == null) {//结尾添加的话在添加完集合元素后将最后一个集合的节点对象pred作为last
        last = pred;
    } else {
        pred.next = succ;//将集合元素的最后一个节点对象的next指针指向原index位置上的Node对象
        succ.prev = pred;//将原index位置上的pred指针对象指向集合的最后一个对象
    }
 
    size += numNew;
    modCount++;
    return true;
}
/**
 * Returns the (non-null) Node at the specified element index.
 * 返回index位置的非空节点
 * 折半查询 
 */
Node<E> node(int index) {
    /**
     * 如果index小于当前元素个数的一半，则从前向后遍历查询 ，否则从后向前遍历查询
     */
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

这里面主要是两个方法：

- addAll(int index, Collection<? extends E> c)，这里面首先是判断了是否会出现索引越界的坑你，然后定义pred和succ两个Node对象，用于标识要插入元素的前置节点和后置节点，这段代码的工作原理，可以理解为一根筷子切成A,B两根,A的末尾处的节点为新插入元素的前置节点，B的开始出的节点为新插入元素的后置节点，新插入的元素集合依次放在A,B之间，然后把前置节点和后置节点连接上，就插入完成了。
- node（int index）:这个方法的主要功能是找到index位置的Node节点，源码上利用折半查询进行优化，即使这样，遍历和查询效率还是比较差。

#### 删除方法

>根据元素移除

```java
/**
 *从第一个节点循环指针查找
 */
public boolean remove(Object o) {
    //如果移除的数据为Null
    if (o == null) {
        //遍历找到第一个为null的节点，然后移除掉
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
    //遍历找到第一条不为null与参数相等的数据，然后移除掉
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
 
E unlink(Node<E> x) {
    // assert x != null;
	//移除的数据
    final E element = x.item;
    //移除节点的后置节点
    final Node<E> next = x.next;
  //移除节点的前置节点
    final Node<E> prev = x.prev;
    
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
 
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
 
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

>根据索引移除

```java
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

#### 获取方法

>get(index),getFirst(),getLast()

```java
public E get(int index) {
    checkElementIndex(index);//检查是否越界
    return node(index).item;//折半查询节点，然后获取该节点的值
}
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

#### 其他方法

>set

```java
public E set(int index, E element) {
    checkElementIndex(index);//检查是否越界
    Node<E> x = node(index);//折半查询索引为index的节点
    E oldVal = x.item;//查询index节点原来的数据值
    x.item = element;//将新值插入
    return oldVal;//返回旧值
}
```

>clear

```java
public void clear() {
    //遍历所以的数据，置为null,方便垃圾回收
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

>toArray

```java
public Object[] toArray() {
    Object[] result = new Object[size];
    int i = 0;
    //遍历所有的节点，将节点中的值放入数组中
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;
    return result;
}
```

### 总结

- LinkedList的实现是基于双向循环链表的，且头结点中不存放数据。
- 在查找和删除某元素时，源码中都划分为该元素为null和不为null两种情况来处理，LinkedList中允许元素为null
- LinkedList是基于链表实现的，因此不存在容量不足的问题，所以这里没有扩容的方法
- LinkedList是基于链表实现的，因此插入删除效率高，查找效率低
- 实现了栈和队列的操作方法，因此也可以作为栈、队列和双端队列来使用