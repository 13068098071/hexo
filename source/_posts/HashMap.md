---
title: HashMap
date: 2021-12-24
description: hashMap集合笔记
cover: https://s2.loli.net/2021/12/23/TNY612AUy38Dfk7.jpg
tags: Map
categories: 集合
---
## HashMap

### 数据结构

 在`JDK1.7`中HashMap的数据结构是`数组 + 链表` , 而在`JDK1.8`中则演化成了`数组 + 链表 + 红黑树`的结构 , 这也是1.8中最大的更新 , 下面我们来探究一下为何要演化为`数组 + 链表 + 红黑树`这样的数据结构

我们知道在1.7中当产生了`hash碰撞`时便会将当前`Entry`变成链表 , 单向链表查找除了`head`节点外的时间复杂度都是`O(n)` , 如果频繁的发生了`hash碰撞`每次查找元素都是非常耗费时间的 , 所以为了避免这一现象1.8中引入了红黑树

红黑树的插入、查找的时间复杂度都是`O(log n)` , 假如你的红黑树里面有256个数据 , 此时只需要8次就能找到目标数据 , 即使是65536个数据也只需要16次即可 , 效率相比链表而言提升的非常大

HashMap转为红黑树后存储的数据结构图

![1.8HashMap存储](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211224152017394.png)

### 源码解析

#### 核心参数

```java
//默认初始化table数组容量16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
  			
//table最大容量1073741824
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认加载因子, 即当现有数组长度达到容量的75%时会进行扩容操作
static final float DEFAULT_LOAD_FACTOR = 0.75f;
  
//1.8新增 当链表的长度 >=8 - 1 时会转换为红黑树, 关于为什么要定义为8的详细解读在下面↓
static final int TREEIFY_THRESHOLD = 8;
  
//1.8新增 当红黑树的长度 <=6 时会转换为链表, 关于为什么红黑树 → 链表的阈值是6的详细解读在下面↓
static final int UNTREEIFY_THRESHOLD = 6;
  
//1.8新增 红黑树的最小容量
static final int MIN_TREEIFY_CAPACITY = 64;
  
//定义一个类型为Node<K,V>的table数组
transient Node<K,V>[] table;
  
//table数组的长度
transient int size;

//实际的扩容的阈值 threshold = 容量 * 加载因子
//在构造器中会被初始化为DEFAULT_INITIAL_CAPACITY的值16
//在第一次存储数据时会在inflateTable()方法中再次赋值threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
int threshold;

//实际的加载因子, 在构造器中进行初始化
//如果创建HashMap时没有指定loadFactor的大小则会初始化为DEFAULT_INITIAL_CAPACITY的值
final float loadFactor;

//HashMap更改的次数
//用来作为并发下判断是否有其它线程修改了该HashMap,抛出ConcurrentModificationException
transient int modCount;

//在初始化时指定初始长度及加载因子的构造器
public HashMap(int initialCapacity, float loadFactor) {
   ...
}

//在初始化时指定初始长度的构造器
public HashMap(int initialCapacity) {
  	//这里调用的其实还是上面的构造器
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

//什么也不指定的构造器 , 这里不像1.7中还是去调用了有参构造器 , 具体原因下面会有分析
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
}
```

>##### `TREEIFY_THRESHOLD`

这个参数是链表转换成红黑树的阈值，TREEIFY_THRESHOLD = 8

1. 为什么不在一开始就使用红黑树来替代链表

   相同数据量下红黑树(TreeNode)占用的空间是链表(Node)的俩倍 , 考虑到时间和空间的权衡 , 只有当链表的长度达到阈值时才会将其转成红黑树

2. 为什么链表 → 红黑树的阈值是8呢?

   ```java
   * In usages with well-distributed user hashCodes, tree bins are
   * rarely used.  Ideally, under random hashCodes, the frequency of
   * nodes in bins follows a Poisson distribution
   * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
   * parameter of about 0.5 on average for the default resizing
   * threshold of 0.75, although with a large variance because of
   * resizing granularity. Ignoring variance, the expected
   * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
   * factorial(k)). The first values are:
   *
   * 0:    0.60653066
   * 1:    0.30326533
   * 2:    0.07581633
   * 3:    0.01263606
   * 4:    0.00157952
   * 5:    0.00015795
   * 6:    0.00001316
   * 7:    0.00000094
   * 8:    0.00000006
   * more: less than 1 in ten million
   ```

   HashMap的作者认为在理想的情况下随机hashCode算法下所有节点的分布频率会遵循[泊松分布(Poisson distribution)](http://en.wikipedia.org/wiki/Poisson_distribution) , 上面也列举了链表长度达到8的概率是0.00000006,也就是说我们几乎不可能会使用到红黑树 , 所以作者使用8作为一个分水岭

>UNTREEIFY_THRESHOLD

为何链表 → 红黑树的阈值，UNTREEIFY_THRESHOLD = 6

1. 为何链表 → 红黑树的阈值是6

   假设UNTREEIFY_THRESHOLD的 = 7 , 当我们有频繁的添加和删除操作时 , hash碰撞产生的节点数量 一旦在7附件徘徊就会造成红黑树和链表的频繁转换 , 此时我们大多数的性能就都耗费在了链表 → 红黑树和红黑树 → 链表` ,这样反而就得不偿失了 , 所以作者将长度为7作为一个缓存地段从而选取了6作为红黑树 → 链表的阈值

>loadFactor

- 加载因子并不是越大越好的 , 虽然加载因子越大就意味着HashMap的实际容量越大 , 扩容的次数越少 , 但是因为实际存储的数据大了 , 俩个相同容量的HashMap加载因子越大的那个读取的速度更慢 , 所以我们需要根据自己的实际使用情况来进行判断 , 是要存储更多的数据呢 , 还是要更快的读取速度
- 加载因子是会影响到扩容的次数的 , 如果加载因子太小的话HashMap会频繁的进行扩容 , 导致在存储的时候性能下降
- 如果我们在创建HashMap时就已经知道了要存储的数据量 , 那么我们完全可以通过实际存储数量 ÷ 0.75来计算出我们初始化的HashMap容量 , 这样可以避免HashMap再进行扩容操作 , 提升代码效率

>modCount

HashMap不是线程安全的 , 也就是说你在操作的同时可能会有其它的线程也在操作该map,那样会造成脏数据 , 所以为了避免这种情况发生HashMap、ArrayList等使用了fail-fast策略 , 用modCount来记录修改集合修改次数

我们在边迭代边删除集合元素时会碰到一个异常ConcurrentModificationException , 原因是不管你使用entrySet()方法也好 , keySet()方法也好 , 其实在for循环的时候还是会使用该集合内置的Iterator迭代器中的nextEntry()方法 , 如果你没有使用Iterator内置的remove()方法 , 那么迭代器内部的记录更改次数的值便不会被同步 , 当你下一次循环时调用nextEntry()方法便会抛出异常

#### put方法

![HashMap的put](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211224153116502.png)

```java
 public V put(K key, V value) {
     return putVal(hash(key) ,  key, value, false, true);
 }
 
 //关于onlyIfAbsent,evict的讲解在下面↓
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                  boolean evict) {
     Node<K,V>[] tab; 
     Node<K,V> p; 
     int n, i;
     // 判断数组是否为空 , 为空则调用resize()方法进行初始化,resize()方法源码在下面↓
     if ((tab = table) == null || (n = tab.length) == 0)
          n = (tab = resize()).length;

 	  // 如果当前数组索引位置的元素为null则直接创建新的节点并存到该索引位置的数组中
 	  // 和JDK1.7的对比
 	  //1. 其实这里的(n - 1) & hash就是JDK1.7中的indexFor()方法体中的(length - 1) & h,只不过在1.8中做了简化处理
      if ((p = tab[i = (n - 1) & hash]) == null)
          //2. tab[i] = newNode(hash, key, value, null);在JDK1.7中时在createEntry()方法中完成创建Entry的 , 在1.8中该方法也被删除了
          tab[i] = newNode(hash, key, value, null);
			
	  // 走到这个else代码块中说明当前key已经在数组中存在了 || 发生了hash碰撞
      else {
          Node<K,V> e; K k;
          // 判断当前key在数组中是否存在, 存在则不进行存储操作
          // 有朋友好奇这里的p是在哪里初始化的, 其实在上一个if里面就被初始化了
          if (p.hash == hash &&
              ((k = p.key) == key || (key != null && key.equals(k))))
              e = p;
        
          // 判断当前节点类型是否是红黑树节点
          else if (p instanceof TreeNode)
              // 将key、value存储到红黑树节点中 , 如果key已经存在 , 则返回之前的节点 , 源码解析在下面↓
              e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        
          // 代码进入到这个else代码块就说明发生了hash碰撞
          else {
              // 该代码块中的e的初始化会有俩种情况
              // ① e是最后一个节点的next元素,e == null
              // ② e是和key相同的那个node
              for (int binCount = 0; ; ++binCount) {
                  // 判断当前节点是否是链表的最后一个节
                  // ①
                  if ((e = p.next) == null) {
                      // 直接将当前要存储的key、value存储到上一个节点的next元素中
                      // 在JDK1.7中的会先将之前存储的节点取出来, 然后将之前的节点作为新节点的next元素存储到数组中
                      // 而在JDK1.8中会直接将新的节点放到之前存储节点的next元素中
                      // 也就是说JDK1.7中的链表插入顺序是从头部开始插入, 而在1.8中时从尾部开始插入
                      p.next = newNode(hash, key, value, null);
                      // 因为这个循环是迭代链表的, 所以binCount代表着链表的长度, 如果链表长度超过阈值则会转换为红黑树
                      if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                          // 转换红黑树的源码解析在下面↓
                          treeifyBin(tab, hash);
                      break;
                  }
                
                  // 当前节点不是最后一个节点, 判断当前节点的key与要存储的key是否相同
                  // ②
                  if (e.hash == hash &&
                      ((k = e.key) == key || (key != null && key.equals(k))))
                      break;
                  p = e;
              }
          }
        
  		  // e在只有当前key已经存在时才会完成初始化], 这里是统一处理, 返回key之前对应的value, 并判断是否要替换value,默认是替换
          if (e != null) { // existing mapping for key
              V oldValue = e.value;
              if (!onlyIfAbsent || oldValue == null)
                  e.value = value;
              // 这里是个空方法体
              afterNodeAccess(e);
              return oldValue;
          }
      }
      ++modCount;
 	  // 在JDK1.7中扩容操作是在存储数据之前发生的
 	  // 在JDK1.8中扩容操作是在存储数据之后发生的
      if (++size > threshold)
          resize();
	  // 这里是个空方法体
      afterNodeInsertion(evict);
      return null;
}
```

- 首先会判断哈希表Node<k,v>[] table是否为空或者null，是则进行resize()方法进行扩容，否则进入下一个阶段；
- 根据key计算的hash值，hash值&（length-1）得到数组所在的存储位置table[i]，如果存储位置没有元素存放，则直接创建一个新Node；
- 如果存储位置有元素存放，则接下来会判断该位置元素的hash值和key值是否和当前操作元素一致，如果一致，说明是修改value操作，直接进行覆盖；
- 当前存储位置有元素，又不和当前操作元素一致，说明发生了hash冲突，在这里他又有两种方式，分别是链表和红黑树，区别就在于他们的头结点，如果头结点是treeNode，则说明此位置结构是红黑树方式插入；
- 如果不是红黑树，则说明是单链表，将新增节点插入至链表的最后位置，但是这里还有一个判断点，判断当前链表长度是否大于等于8.如果是则将链表转化为红黑树，遍历过程如果key存在，则直接覆盖；
- 最后就是判断当前存储键值对的数量，如果大于阈值值，则扩容；

>onlyIfAbsent

onlyIfAbsent：如果为true，则不更改现有值 , 也就是说不会用新的value来替换旧的value

>##### `hash()`

```java
static final int hash(Object key) {
    int h;
    //高16位和低16位进行异或运算，增散列程度，减少hash碰撞发生的概率
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

>**为什么HashMap的容量一定要是2的非零次方幂而且还要进行`hash & (length - 1)`的操作**

如果容量是2的次方 , 那么length - 1得到的二进制的除了补位外都是1,根据&运算符的规则 , 0&0=0; 0&1=0; 1&1=1;那么也就意味着不论hash的值是什么 , 只要length - 1的二进制码是这样规律的 , 那么就可以保证hash的值只有和length - 1的同位参与了运算 , 例如二进制码A(10101011)&B(00001111)的结果就是C(00001011) , C的结果只会受到B二进制码后四位的影响 , 因为b的补位都是0 , 也就是说h & (length - 1)得到的索引不会大于length,也就不会越界

#### resize方法

```java
final Node<K,V>[] resize() {
   Node<K,V>[] oldTab = table;
   int oldCap = (oldTab == null) ? 0 : oldTab.length;
   int oldThr = threshold;
   int newCap, newThr = 0;
   // 进入到这个if代码块中说明此时table数组已经完成过初始化
   if (oldCap > 0) {
       // 判断当前容量是否已达到最大值
       if (oldCap >= MAXIMUM_CAPACITY) {
           threshold = Integer.MAX_VALUE;
           return oldTab;
       }
     
       // 将新数组的容量设置为旧容量的2倍
       else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
           // 这一步的操作等同于 oldThr * 2
           newThr = oldThr << 1; // double threshold
   }

   // 进入到这个代码块说明在创建HashMap时使用的有参构造器
   // 在翻阅代码后我们发现threshold的值会在有参构造器中被初始化 , 此时被初始化了就会使用指定的容量来完成table数组的初始化
   else if (oldThr > 0) // initial capacity was placed in threshold
       newCap = oldThr;

   // 进入到这个else代码块中说明数组还未完成初始化 , 进行初始化操作
   else {               
       newCap = DEFAULT_INITIAL_CAPACITY;
       newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
   }
 		
   // 只有调用有参构造器才让newThr == 0,至于为什么不放到上面的else if中 , 可能是为以后扩展做铺垫吧
   if (newThr == 0) {
       float ft = (float)newCap * loadFactor;
       newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                 (int)ft : Integer.MAX_VALUE);
   }
   threshold = newThr;
   @SuppressWarnings({"rawtypes" , "unchecked"})
   // 初始化新数组
   Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
   table = newTab;
   if (oldTab != null) {
       for (int j = 0; j < oldCap; ++j) {
           Node<K,V> e;
           if ((e = oldTab[j]) != null) {
               oldTab[j] = null;
               // 如果当前不是链表则会直接将当前元素放到新数组中旧元素所在索引位置
               // 至于为什么 , 在下面会有详细讲解↓
               if (e.next == null)
                   newTab[e.hash & (newCap - 1)] = e;
             	
               // 如果当前元素是红黑树的节点
               else if (e instanceof TreeNode)
                   ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
             
               // 进入这个代码块中说明当前数组元素是链表
               else {
                   // loHead和loTail存储的是转移数据后仍然存储在当前索引位置的元素
                   Node<K,V> loHead = null, loTail = null;
                   // hiHead和hiTail存储的是转移数据后存储到[j + oldCap]索引位置的元素
                   // 这里看不懂没事 , 接着往下看就明白了
                   Node<K,V> hiHead = null, hiTail = null;
                   Node<K,V> next;
                 e = 1 e.next = 2 e.next.next = 3

                   do {
                       // 为了方便讲明白这个循环里面的源码,我们来举个例子,假设此时链表有3个元素a,b,c
                       // 第一次循环进入到① | ②
                       // tail元素肯定为null,所以会将当前e赋值给head元素,并且为tail赋值,以便下次循环
                       // 此时head元素会含有e的链表关系即next元素指针,此时就是e=a  head=a  tail=a
                       
                       // 第二次循环进入① | ②
                       // 还是会通过第一次进入时的索引判断 , 所以此时tail元素不会为null,
                       // 此时就是 e=b  head=a  head.next=b  tail=b  tail.next=b
                       // 因为在第一次进入的时候head和tail是同时指向e的
                       // 所以此时tail.next=b也就意味着head.next=b,所以才会先在else代码块中完成tail.next的初始化
                       // 再完成tail的初始化
                       
                   	   // 第三次循环进入① | ② 就是 e=c  head=a  tail=c  tail.next=c  head.next.next=c
                   	   // 以此类推....
                       next = e.next;
                       // 这里会根据(e.hash & oldCap) == 0来将链表划分为俩部分 , 一部分仍然存储在旧链表的索引位置 , 另一部分存储到新数组的[j + oldCap]索引位置
                       if ((e.hash & oldCap) == 0) {
                           // ①
                           if (loTail == null)
                               loHead = e;
                           // ②
                           else
                               loTail.next = e;
                           loTail = e;
                       }
                       else {
                           // ①
                           if (hiTail == null)
                               hiHead = e;
                           // ②
                           else
                               hiTail.next = e;
                           hiTail = e;
                       }
                   } while ((e = next) != null);
                   // 将loHead存储到新数组的旧索引位置
                   if (loTail != null) {
                       loTail.next = null;
                       newTab[j] = loHead;
                   }
                   // 将hiHead存储到新数组的[j + oldCap]索引位置
                   if (hiTail != null) {
                       hiTail.next = null;
                       newTab[j + oldCap] = hiHead;
                   }
               }
           }
       }
   }
   return newTab;
}
```

#### get方法

![HashMap的get](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211224154248251.png)

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key) ,  key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 计算索引位置并判断当前索引位置是否存在元素
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 判断当前元素是否与要取得值相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 判断当前元素是否有next元素
        if ((e = first.next) != null) {
            // 判断当前元素是否是红黑树节点
            if (first instanceof TreeNode)
                // 寻找红黑树中与要取值相等的节点
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 迭代链表
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### 1.7和1.8对比

|              | 1.7                            | 1.8                               |
| ------------ | ------------------------------ | --------------------------------- |
| 数据结构     | 数组 + 链表                    | 数组 + 链表 + 红黑树              |
| 节点         | Entry                          | Node TreeNode                     |
| Hash算法     | 较为复杂                       | 异或hash右移16位                  |
| 对Null的处理 | 单独写一个putForNull()方法处理 | 作为以一个Hash值为0的普通节点处理 |
| 初始化       | 赋值给一个空数组，put时初始化  | 没有赋值，懒加载，put时初始化     |
| 扩容         | 插入前扩容                     | 插入后，初始化，树化时扩容        |
| 节点插入     | 头插法                         | 尾插法                            |

参看文档：https://blog.csdn.net/weixin_44141495/article/details/108402128

### 常见问题

>**什么时候扩容**

1. 当前容量超过阈值
2. 当链表中元素个数超过默认设定（8个），当数组的大小还未超过64的时候，此时进行数组的扩容，如果超过则将链表转化成红黑树

>**什么时候链表转化为红黑树**

当数组大小已经超过64并且链表中的元素个数超过默认设定（8个）时，将链表转化为红黑树

> **为什么HashMap的初始容量是16，扩容时一定要是2的n次方**

index = HashCode（key） & (length-1)

以值为“book”的Key来演示：

1. 计算book的hashcode，结果为十进制的3029737，二进制的101110001110101110 1001。
2. 假定HashMap长度是默认的16，计算Length-1的结果为十进制的15，二进制的1111。
3. 把以上两个结果做与运算，101110001110101110 1001 & 1111 = 1001，十进制是9，所以 index=9。

**Hash算法最终得到的index结果，完全取决于Key的Hashcode值的最后几位**

假设HashMap的长度是10，重复刚才的运算步骤：

![img](https://img-blog.csdnimg.cn/20190109165033757.jpg)

单独看这个结果，表面上并没有问题。我们再来尝试一个新的HashCode 101110001110101110 **1011** ：

![img](https://img-blog.csdnimg.cn/20190109165144678.jpg)

虽然HashCode的倒数第二第三位从0变成了1，但是运算的结果都是1001。也就是说，当HashMap长度为10的时候，有些index结果的出现几率会更大，而有些index结果永远不会出现.这样，显然不符合Hash算法均匀分布的原则。

反观长度16或者其他2的幂，Length-1的值是所有二进制位全为1，这种情况下，index的结果等同于HashCode后几位的值。只要输入的HashCode本身分布均匀，Hash算法的结果就是均匀的。

**总结**

- 取模运算效率很低，为了实现高效的Hash算法，HashMap的采用了位运算的方式
- 为了Hash算法的结果就是均匀的

> 深入理解HashMap线程不安全的体现

https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653192000&idx=1&sn=118cee6d1c67e7b8e4f762af3e61643e&chksm=8c990d9abbee848c739aeaf25893ae4382eca90642f65fc9b8eb76d58d6e7adebe65da03f80d&scene=21#wechat_redirect

> 如何处理hash冲突

- 开发定址法：既然当前位置容不下冲突的元素了，那就再找一个空的位置存储 Hash 冲突的值（当前 index 冲突了，那么将冲突的元素放在 index+1)。

- 再散列法：换一个 Hash 算法再计算一个 hash 值，如果不冲突了就存储值（例如第一个算法是名字的首字母的 Hash 值，如果冲突了，计算名字的第二个字母的 Hash 值，如果冲突解决了则将值放入数组中）。

- 链地址法：每个数组中都存有一个单链表，发生 Hash 冲突时，只是将冲突的 value 当作新节点插入到链表（HashMap 解决冲突的办法）。

- 公共溢出区法：将冲突的 value 都存到另外一个顺序表中，查找时如果当前表没有对应值，则去溢出区进行顺序查找。
  