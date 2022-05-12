---
title: ArrayList
date: 2021-12-24
description: ArrayList集合笔记
cover: https://s2.loli.net/2021/12/23/TNY612AUy38Dfk7.jpg
tags: List
categories: 集合
---


## ArrayList

### 类图

![类图](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211224105430699.png)

`ArrayList`继承于 `AbstractList`，实现了 `List`, `RandomAccess`, `Cloneable`, `java.io.Serializable` 这些接口。

- `RandomAccess`是一个标志接口，表明实现这个这个接口的 `List`集合是支持快速随机访问的
- 实现 `Cloneable`接口并覆盖了方法`clone()`，能被克隆
- 实现了java.io.Serializable 接口，这意味着`ArrayList`支持序列化，能通过序列化去传输

### 源码分析

#### 成员变量

```java
private int size;  // 实际元素个数
transient Object[] elementData; //真正保存元素的数组
private static final int DEFAULT_CAPACITY = 10;//默认的初始容量大小
```

#### 构造方法

无参数直接初始化、指定大小初始化、指定初始数据初始化

```java
//1、无参数直接初始化，数组大小为空
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
//2、指定初始数据初始化
public ArrayList(Collection<? extends E> c){
    //elementData是保存数组的容器，默认为null
    elementData=c.toArray();
    //如果给定的集合（c)数据有值
    if((size=elementData.length)!=0){
      	//c.toArray might(incorrectly)not return Object[](see 6260652)
      	//如果集合元素类型不是Object类型，我们会转成Object
	    if(elementData.getClass()!=Object[].class){
	        elementData=Arrays.copyOf(elementData,size,Object].class);
	    }
  	}else{
    	//给定集合（c)无值，则默认空数组
    	this.elementData=EMPTY_ELEMENTDATA
  	}
}
//3、指定初始容量
public ArrayList(int initialCapacity) {
	//指定的初始容量大于0，将elementData初始化为指定大小的数组
   if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
    	//否则初始化成一个空数组
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

>补充

- `ArrayList`无参构造器初始化时，默认大小是空数组，并不是大家常说的10,10是在第一次`add`的时候扩容的数组值

- 使用方式二进行创建对象时，如果入参容器保存的对象不是`Object`，则转换为`Object`

- DEFAULTCAPACITY_EMPTY_ELEMENTDATA`和`EMPTY_ELEMENTDATA是啥：它其实是定义在成员变量的两个空数组

  ```java
  private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
  private static final Object[] EMPTY_ELEMENTDATA = {};
  ```

#### 新增和扩容实现

新增时就是给数组中添加元素，主要分为两步走：

- 判断是否需要扩容，如果需要扩容执行扩容操作
- 直接赋值

>新增源码

```java
public boolean add(E e) {
	//确保数组大小是否足够，不够执行扩容，size为当前数组元素个数，判断size+1是因为后面还要size++
    ensureCapacityInternal(size + 1);  //1
    elementData[size++] = e;//2
    return true;
}
```

>扩容部分源码

```java
private void ensureCapacityInternal(int minCapacity) {
	//先调用calculateCapacity计算容量
  	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
private static int calculateCapacity(Object[] elementData, int minCapacity) {
  //如果当前数组还是个空数组，也就是他用过无参构造去初始化的
  //那么直接返回DEFAULT_CAPACITY，即10
  if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 如果当前容量已经大于当前数组的长度了，说明需要去扩容了
    if (minCapacity - elementData.length > 0)
        //扩容
        grow(minCapacity);
}

private void grow(int minCapacity){
  int oldCapacity = elementData.length;
  //oldCapacity>>1是把oldCapacity除以2的意思
  int newCapacity=oldCapacity+(oldCapacity>>1);
  //如果扩容后的值<我们的期望值，扩容后的值就等于我们的期望值
  if(newCapacity-minCapacity<0)
    newCapacity = minCapacity;
  //如果扩容后的值>jvm所能分配的数组的最大值，那么就用Integer的最大值
  if(newCapacity-MAX_ARRAY_SIZE>0)
    elementData=Arrays.copyOf(elementData,newCapacity);
}
```

>注意

- 新增时，没有对值进行校验，所以新增值可以为`null`，且没有做重复值判断，所以元素可以`重复`
- ArrayList中的数组的最大值是`Integer.MAX_VALUE`，超过这个值，`JVM`就不会给数组分配内存空间了
- 扩容是原来容量大小+容量大小的一半，简单说就是扩容后的大小是原来容量的1.5倍
- 扩容完成之后，就是简单的赋值了，赋值时并没有加锁，所以是线程`不安全`的

#### 扩容的本质

在`grow`方法的最后，扩容是通过`Arrays.copyOf(elementData,newCapacity);`这行代码实现的。这个方法实际上调用的方法是我们经常使用的`System.arraycopy`：

```java
/**
*@param src 被拷贝的数组
*@param srcPos 从数组那里开始
*@param dest 目标数组
*@param destPos从目标数组那个索引位置开始拷贝
*@param length 拷贝的长度
*此方法是没有返回值的，通过dest的引用进行传值
*/
public static native void arraycopy(Object src, int srcPos,Object dest, int destPos,int length);
```

这个方法是一个native方法，虽然不能看到方法内部的具体实现，但通过参数也可以管中窥豹。这个方法会移动元素。所以说数组如果要扩容，需要重新分配一块更大的空间，再把数据全部复制过去，时间复杂度 O(N)；而且你如果想在数组中间进行插入和删除，每次必须搬移后面的所有数据以保持连续，时间复杂度 O(N)。由于数组又是一块连续的内存空间，能够根据索引快速访问元素。
上面也就解释了一开始那个问题：ArrayList为什么插入慢，查询快。

#### 删除

`ArrayList`有多种删除方法，这里以根据值删除的方式进行说明(其他原理类似)

```java
public boolean remove(Object o) {
  //如果要删除的值是null,删除第一个是null的值
  if(o==null){
    for(int index=0;index<size;index++)
      if(elementData[index]==null){
        fastRemove(index)
        return true;
      }
  }else{
    //如果要删除的值不为null,找到第一个和要删除的值相等的删除
    for(int index=0;index<size;index++)
      //这里是根据 equals来判断值相等的，相等后再根据索引位置进行删除
      //所以根据对象删除时，一般来说，如果你确定要删除的是某一类的业务对象，则需要重写equals
      if(o.equals(elementData[index]){
        fastRemove(index)
        return true;
      }
  }
  return false
}
```

核心其实是`fastRemove`方法

```java
private void fastRemove(int index){
  //记录数组的结构要发生变动了
  nodCount++;
  //numMoved表示删除index位置的元素后，需要从index后移动多少个元素到前面去
  //减1的原因，是因为size从1开始算起，index从0开始算起
  int numMoved=size-index-1;
  if(numMoved>0)
    //从index+1位置开始被拷贝，拷贝的起始位置是index,长度是numMoved
    System.arraycopy(elementData, index+1, elementData, index, numMoved);
  //数组最后一个位置赋值null,帮助GC(没有引用则自动回收了)
  elementData[--size] = null;
}
```

从源码中，我们可以看出，某一个元素被删除后，为了维护数组结构，我们都会把数组后面的元素往前移动，同时释放最后一个引用，便于回收。

### 迭代器iterator

在用 for 遍历集合的时候是不可以对集合进行 remove操作的，因为 remove 操作会改变集合的大小。从而容易造成结果不准确甚至数组下标越界，更严重者还会抛出 ConcurrentModificationException。

#### 并发修改异常

```java
package com.ma.collection.arraylist;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 马志超
 * @title: TestArrayList01
 * @projectName login
 * @description: TODO
 * @date 2021/12/24 11:18
 */
public class TestArrayList01 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add("c");
        list.add("d");
        list.add("e");
        for (String s : list) {
            if("a".equals(s)){
                list.remove(s);
            }
        }
    }
}
```

![并发修改异常](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211224112032656.png)

产生这样的异常的原因是：对集合元素进行一次修改时，对应的modCount的值就会加一，导致expectedModCount 与 modCount 不相等，然后抛出ConcurrentModificationException 。调用迭代器的删除方法，会将修改后的modCount的值赋值给expectedModCount

#### 迭代器的源码

```java
private class Itr implements Iterator<E> {
	int cursor;       // 代表下一个要访问的元素下标
	int lastRet = -1; // 代表上一个要访问的元素下标
	int expectedModCount = modCount; //代表对 ArrayList 修改次数的期望值，初始值为 modCount
	
    //如果下一个元素的下标等于集合的大小 ，就证明到最后了。
	public boolean hasNext() {
		return cursor != size;
	}

	@SuppressWarnings("unchecked")
	public E next() {
        //首先判断 expectedModCount 和 modCount 是否相等
		checkForComodification();
		int i = cursor;
        //看是否超过集合大小和数组长度
		if (i >= size)
			throw new NoSuchElementException();
		Object[] elementData = ArrayList.this.elementData;
		if (i >= elementData.length)
			throw new ConcurrentModificationException();
        //将 cursor 自增 1
		cursor = i + 1;
		return (E) elementData[lastRet = i];
	}

	public void remove() {
		if (lastRet < 0)
			throw new IllegalStateException();
		checkForComodification();

		try {
			ArrayList.this.remove(lastRet);
			cursor = lastRet;
			lastRet = -1;
			expectedModCount = modCount;
		} catch (IndexOutOfBoundsException ex) {
			throw new ConcurrentModificationException();
		}
	}

	final void checkForComodification() {
		if (modCount != expectedModCount)
			throw new ConcurrentModificationException();
	}
}
```

>hasNext

如果下一个元素的下标等于集合的大小 ，就证明到最后了

>next

首先判断 `expectedModCount` 和 `modCount` 是否相等。然后对 `cursor` 进行判断，看是否超过集合大小和数组长度。然后将 `cursor` 赋值给 `lastRet` ，并返回下标为 lastRet 的元素。最后将 cursor 自增 1。开始时，`cursor = 0，lastRet = -1`；每调用一次 next 方法， cursor 和 lastRet 都会自增 1

>remove

首先会判断 `lastRet` 的值是否小于 0，然后在检查 `expectedModCount` 和 `modCount` 是否相等。接下来是关键，直接调用 ArrayList 的 `remove` 方法删除下标为 `lastRet` 的元素。然后将 `lastRet` 赋值给 cursor ，将 `lastRet` 重新赋值为 -1，并将 `modCount` 重新赋值给 `expectedModCount`

>异常解决

```java
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String value = iterator.next();
    if ("a".equals(value)) {
        iterator.remove();
    }
}
```

直接调用 iterator.remove() 即可。因为在该方法中增加了 expectedModCount = modCount 操作

>弊端

- 只能进行remove操作，add、clear 等 Itr 中没有
- 调用 remove 之前必须先调用 next。因为 remove 开始就对 lastRet 做了校验。而 lastRet 初始化时为 -1
- next 之后只可以调用一次 remove。因为 remove 会将 lastRet 重新初始化为 -1

### 总结

- ArrayList 底层基于数组实现容量大小动态可变。
-  扩容机制为首先扩容为原始容量的 1.5 倍。如果1.5倍太小的话，则将我们所需的容量大小赋值给 newCapacity，如果1.5倍太大或者我们需要的容量太大，那就直接拿 `newCapacity = (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE` 来扩容。 扩容之后是通过数组的拷贝来确保元素的准确性的，所以尽可能减少扩容操作。 
- ArrayList 的最大存储能力：Integer.MAX_VALUE。 
- size 为集合中存储的元素的个数。
- elementData.length 为数组长度，表示最多可以存储多少个元素。 
- 如果需要边遍历边 remove ，必须使用 iterator。且 remove 之前必须先 next，next 之后只能用一次 remove。