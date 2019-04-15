# Java集合知识

##  常见的集合

Map 接口和 Collection 接口是所有集合框架的父接口：

1. Collection 接口的子接口包括：Set 接口和 List 接口；

2. Map 接口的实现类主要有：HashMap、TreeMap、Hashtable、ConcurrentHashMap 以及 Properties 等；

3. Set 接口的实现类主要有：HashSet、TreeSet、LinkedHashSet 等；

4. List 接口的实现类主要有：ArrayList、LinkedList、Stack 以及 Vector 等
##  List、Set 和 Map 的初始容量和加载因子

1. List

ArrayList 的初始容量是 10；加载因子为 0.5； 扩容增量：原容量的 0.5 倍 +1；一次扩容后长度为 16。

Vector 初始容量为 10，加载因子是 1。扩容增量：原容量的 1 倍，如 Vector 的容量为 10，一次扩容后是容量为 20。

2. Set

HashSet，初始容量为 16，加载因子为 0.75； 扩容增量：原容量的 1 倍； 如 HashSet 的容量为 16，一次扩容后容量为 32

3. Map

HashMap，初始容量 16，加载因子为 0.75； 扩容增量：原容量的 1 倍； 如 HashMap 的容量为 16，一次扩容后容量为 32

## 迭代器



**快速失败**（fail-fast）

在使用迭代器对集合对象进行遍历的时候，如果 A 线程正在对集合进行遍历，此时 B 线程对集合进行修改（增加、删除、修改），或者 A 线程在遍历过程中对集合进行修改，都会导致 A 线程抛出 ConcurrentModificationException 异常。

**安全失败**（fail-safe）

明白了什么是快速失败之后，安全失败也是非常好理解的。

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，故不会抛 ConcurrentModificationException 异常

## Hashmap（快速失败）

结构同hashtable

**重写equals方法和hashcode方法时，equals方法中用到的成员变量也必定会在hashcode方法中用到,只不过前者作为比较项，后者作为生成摘要的信息项，本质上所用到的数据是一样的，从而保证二者的一致性**

1、如果两个对象相同，那么它们的hashCode值一定要相同；

2、如果两个对象的hashCode相同，它们并不一定相同（上面说的对象相同指的是用eqauls方法比较。）  

**HashMap 的长度为什么是 2 的幂次方？**

通过将 Key 的 hash 值与 length-1 进行 & 运算，实现了当前 Key 的定位，2 的幂次方可以减少冲突（碰撞）的次数，提高 HashMap 查询效率；

如果 length 为 2 的次幂  则 length-1 转化为二进制必定是 11111……的形式，在于 h 的二进制与操作效率会非常的快，而且空间不浪费；

如果 length 不是 2 的次幂，比如 length 为 15，则 length-1 为 14，对应的二进制为 1110，在于 h 与操作，最后一位都为 0，而 0001，0011，0101，1001，1011，0111，1101 这几个位置永远都不能存放元素了，空间浪费相当大。

更糟的是这种情况中，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率！这样就会造成空间的浪费。

## Hashtable(快速失败)

### 成员变量

Hashtable是通过"拉链法"实现的哈希表。它包括几个重要的成员变量：table, count, threshold, loadFactor, modCount。

- table是一个 Entry[] 数组类型，而 Entry（在 HashMap 中有讲解过）实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。

```java
static class Entry<K,V> implements Map.Entry<K,V> {

  final K key;// map中key值，可以为null。

  V value; // map中的value值，可以为null。

  Entry<K,V> next;// 链表引用，防止key值不同，hash值相同。

  int hash; // 每个key的hash值
```

- count 是 Hashtable 的大小，它是 Hashtable 保存的键值对的数量。
- threshold 是 Hashtable 的阈值，用于判断是否需要调整 Hashtable 的容量。threshold 的值="容量*加载因子"。
- loadFactor 就是加载因子。
- modCount 是用来实现 fail-fast 机制的。

默认构造函数：容量 11 加载因子 0.75

### put

首先，通过hash方法计算key的哈希值，并计算得出index值，确定其在table[]中的位置
其次，迭代index索引位置的链表，如果该位置处的链表存在相同的key，则替换value，返回旧的value

### Hashtable 遍历方式

Hashtable 有多种遍历方式：

```java
//1、使用keys()
Enumeration<String> en1 = table.keys();
    while(en1.hasMoreElements()) {
    en1.nextElement();
}

//2、使用elements()
Enumeration<String> en2 = table.elements();
    while(en2.hasMoreElements()) {
    en2.nextElement();
}

//3、使用keySet()
Iterator<String> it1 = table.keySet().iterator();
    while(it1.hasNext()) {
    it1.next();
}

//4、使用entrySet()
Iterator<Entry<String, String>> it2 = table.entrySet().iterator();
    while(it2.hasNext()) {
    it2.next();
}
```

### Hashtable 与 HashMap 的简单比较

1. HashTable 基于 Dictionary 类，而 HashMap 是基于 AbstractMap。Dictionary 是任何可将键映射到相应值的类的抽象父类，而 AbstractMap 是基于 Map 接口的实现，它以最大限度地减少实现此接口所需的工作。

2. HashMap 的 key 和 value 都允许为 null，而 Hashtable 的 key 和 value 都不允许为 null。HashMap 遇到 key 为 null 的时候，调用 putForNullKey 方法进行处理，而对 value 没有处理；Hashtable遇到 null，直接返回 NullPointerException。

3. Hashtable 方法是同步，而HashMap则不是。我们可以看一下源码，Hashtable 中的几乎所有的 public 的方法都是 synchronized 的，而有些方法也是在内部通过 synchronized 代码块来实现。

   所以有人一般都建议如果是涉及到多线程同步时采用 HashTable，没有涉及就采用 HashMap，但是在 Collections 类中存在一个静态方法：synchronizedMap()，该方法创建了一个线程安全的 Map 对象，并把它作为一个封装的对象来返回。

4. Hashtable rehash 大小为2 * 旧容量+1  hashmap rehash 大小为2 * 旧容量

## ConcurrentHashMap（安全失败）

ConcurrentHashMap 类中包含两个静态内部类 HashEntry 和 Segment。HashEntry 用来封装映射表的键 / 值对；Segment 用来充当锁的角色，每个 Segment 对象守护整个散列映射表的若干个桶。每个桶是由若干个 HashEntry 对象链接起来的链表。一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组。

### HashEntry 类

```java
static final class HashEntry<K,V> { 
       final K key;                       // 声明 key 为 final 型
       final int hash;                   // 声明 hash 值为 final 型 
       volatile V value;                 // 声明 value 为 volatile 型
       final HashEntry<K,V> next;
```
### Segment类
Segment 类继承于 ReentrantLock 类，从而使得 Segment 对象能充当锁的角色。每个 Segment 对象用来守护其（成员对象 table 中）包含的若干个桶。

table 是一个由 HashEntry 对象组成的数组。table 数组的每一个数组成员就是散列映射表的一个桶。

count 变量是一个计数器，它表示每个 Segment 对象管理的 table 数组（若干个 HashEntry 组成的链表）包含的 HashEntry 对象的个数。每一个 Segment 对象都有一个 count 对象来表示本 Segment 中包含的 HashEntry 对象的总数。注意，之所以在每个 Segment 对象中包含一个计数器，而不是在 `ConcurrentHashMap 中使用全局的计数器，是为了避免出现“热点域（某个区域并发过高）”而影响 ConcurrentHashMap 的并发性。`

**Java中transient关键字的作用，简单地说，就是让某些被修饰的成员属性变量不被序列化**

### ConcurrentHashMap 类

ConcurrentHashMap 在默认并发级别会创建包含 16 个 Segment 对象的数组。每个 Segment 的成员对象 table 包含若干个散列表的桶。每个桶是由 HashEntry 链接起来的一个链表。如果键能均匀散列，每个 Segment 大约守护整个散列表中桶总数的 1/16。

* 1.8  Synchronized + CAS(Node) 先用链表，当链表长度超过阈值（8）时，转换为红黑树

* ```java
   static class Node<K,V> {
          final int hash;
          final K key;
          volatile V val;
          volatile Node<K,V> next;
   }
  ```

* 1.7 Segment(HashEntry)

**描述一下ConcurrentHashMap中remove操作，有什么需要注意的？**

需要注意如下几点。

第一，当要删除的结点存在时，删除的最后一步操作要将count的值减一。这必须是最后一步操作，否则读取操作可能看不到之前对段所做的结构性修改。

第二，remove执行的开始就将table赋给一个局部变量tab，这是因为table是volatile变量，读写volatile变量的开销很大。编译器也不能对volatile变量的读写做任何优化，直接多次访问非volatile实例变量没有多大影响，编译器会做相应优化。

**结构性修改操作包括 put，remove，clear。下面我们分别分析这三个操作。**

* clear 操作只是把 ConcurrentHashMap 中所有的桶“置空”，每个桶之前引用的链表依然存在，只是桶不再引用到这些链表（所有链表的结构并没有被修改）。正在遍历某个链表的读线程依然可以正常执行对该链表的遍历。

* put 操作如果需要插入一个新节点到链表中时 , 会在链表头部插入这个新节点。此时，链表中的原有节点的链接并没有被修改。也就是说：插入新健 / 值对到链表中的操作不会影响读线程正常遍历这个链表。

* `先根据散列码找到具体的链表；然后遍历这个链表找到要删除的节点；最后把待删除节点之后的所有节点原样保留在新链表中，把待删除节点之前的每个节点克隆到新链表中。下面通过图例来说明 remove 操作。`假设写线程执行 remove 操作，要删除链表的 C 节点，另一个读线程同时正在遍历这个链表。

  ![image007](.\image007.jpg)

![image008](.\image008.jpg)

删除节点 C 之后的所有节点原样保留到新链表中；删除节点 C 之前的每个节点被克隆到新链表中，*注意：它们在新链表中的链接顺序被反转了*。

在执行 remove 操作时，原始链表并没有被修改，也就是说：读线程不会受同时执行 remove 操作的并发写线程的干扰。

综合上面的分析我们可以看出，写线程对某个链表的结构性修改不会影响其他的并发读线程对这个链表的遍历访问。

**ConcurrentHashMap 的高并发性主要来自于三个方面：**

1. 用分离锁实现多个线程间的更深层次的共享访问。
2. 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
3. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

## sychronizedMap

**java.util.Collections.synchronizedMap()**

- 同步整个对象
- 每一次的读写操作都需要加锁
- 对整个对象加锁会极大降低性能
- 这相当于只允许同一时间内至多一个线程操作整个Map，而其他线程必须等待
- 它有可能造成资源冲突（某些线程等待较长时间）
- `SynchronizedHashMap`会返回`Iterator`，当遍历时进行修改会抛出异常

## TreeMap

**数据结构：**

TreeMap基于**红黑树（Red-Black tree）实现**。该映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的 Comparator 进行排序**，具体取决于使用的构造方法。

Entry包含了6个部分内容：key(键)、value(值)、left(左孩子)、right(右孩子)、parent(父节点)、color(颜色)
Entry节点根据key进行排序，Entry节点包含的内容为value。

TreeMap 是一个有序的key-value集合，它是通过红黑树实现的。 
TreeMap 继承于AbstractMap，所以它是一个Map，即一个key-value集合。 
TreeMap 实现了NavigableMap接口，意味着它支持一系列的导航方法。比如返回有序的key集合。 
TreeMap 实现了Cloneable接口，意味着它能被克隆。

TreeMap基于红黑树（Red-Black tree）实现。该映射根据其键的自然顺序(字母排序)进行排序，或者根据创建映射时提供的 Comparator 进行排序，具体取决于使用的构造方法。 
TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。 
TreeMap是非线程安全的。 它的iterator 方法返回的迭代器是fail-fast的。

**HashMap**	                                              **TreeMap**
遍历出来数据无序	                                自然排序或者创建映射提供的Comparator 进行排序
基于散列表	                                           红黑树
取值速度快	                                           取值速度慢

适用于在Map中插入、删除和定位元素	适用于按自然顺序或自定义顺序遍历键(key)

## HashSet

**HashSet**是为独立元素的存放而设计的哈希存储，优点是快速存取。

**HashSet的设计较为“偷懒”，其直接在HashMap*****上封装而成**

由于Set只使用到了HashMap的key，所以此处定义一个静态的常量Object类，来充当HashMap的value

从根源上避免NullPointerException的出现，所以value不设置为null

## TreeSet

和hashset一样，基于TreeMap

