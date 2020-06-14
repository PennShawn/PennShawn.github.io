---
title: HashMap源码学习
categories: 
 - 技术
tags:
 - Java
 - 数据结构
 - 源码
---

HashMap在内部实际上还是以一个Node<K,V>[]的结构存储数据,只不过里面的Node结点可以指向下一个Node,因此可以构成链表或者树的结构.
```
    /* ---------------- Fields -------------- */

    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;

```
HashMap的一些默认值:
```
 /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```
可以看出 
 - hashMap的默认初始容量为16;
 - 最大容量为`1 << 30`;
 - 默认负载因子为`DEFAULT_LOAD_FACTOR = 0.75f`;
 - The bin count threshold for using a tree rather than list for a bin.`static final int TREEIFY_THRESHOLD = 8;`
 - The bin count threshold for untreeifying a (split) bin during a resize operation.`static final int UNTREEIFY_THRESHOLD = 6;`
 - The smallest table capacity for which bins may be treeified.` static final int MIN_TREEIFY_CAPACITY = 64;`<br>
其中, 容量时通过`table`的大小计算的:
```
final int capacity() {
        return (table != null) ? table.length :
            (threshold > 0) ? threshold :
            DEFAULT_INITIAL_CAPACITY;
    }
```



 查看put方法:
```
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
 public V putIfAbsent(K key, V value) {
        return putVal(hash(key), key, value, true, true);
    }
```
可以看到,最后都要调用`putVal`方法:
```
/**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
从上面的代码中我们可以看出hashMap添加节点时的扩容策略和`treeifyBin`策略.
 
 - 扩容<br>
 ```
 if (++size > threshold)
            resize();
 ```
 如果新节点的`key`之前在`map`里不存在,那么`size`就会`+1`,当`size`大于`threshold`之后,会调用`resize()`方法.
 <br>之前看网上分析都是在size大于容量乘以loadFactor后就开始扩容,这里直接比较的是size和threshold的大小.似乎有一点不一样,继续看resize()的代码:
 ```
     final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        ...
    }
 ```
 我们可以看到,当map里面添加第一个元素时:
 ```
  newCap = DEFAULT_INITIAL_CAPACITY;     // 容量
  newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); //threshold
 ```
 将容量设置成`DEFAULT_INITIAL_CAPACITY`,`threshold`设置成 `(int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);`
 之后,每次容量和threshold都是成倍增加,因此:<br>
 `容量 = threshold * loadFactor `,扩容判断条件和网上分析的是一样的.<br>
 也说明了hashMap和Arrlist一样,一开始都是用的一个空数字,在加入第一个元素的时候,才会初始化数组大小.
 
 
  - hash冲突时的处理方法
 把新元素加到table数组中时,如果加入位置上已有元素,如果两者key相同,则用新value覆盖旧value.否则,会将新元素加到该位置上的树或者链表上:
```
 if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //判断是否需要把链表转成红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
```
当链表上`Node<K<V>`的数量大于或等于`TREEIFY_THRESHOLD`时,链表就换转换成红黑树.链表上的结点转换成`TreeNode<K,V>`.

`LinkedHashMap`的有序性是因为它的`node`继承了`HashMap`的`node`,多加了`before`和`after`属性,构成了一条双向链表:
```
 /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

#### hashMap中key的位置是怎么算的?
首先是计算key的hash值
```
 static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
这里`key.hashCode()`之后为什么还要对高16位取抑或运算呢?<br>
先看下一步取位置
```
tab[i = (n - 1) & hash])
```
这里取`(n-1) & hash`很巧妙,因为n是2的整次幂,所以`n-1`比如15二进制表示就是`1111`,和`hash`的`&`运算就相当于是取模了,但是又比`%`运算快.<br>
但是有一个问题,hash的高位没有用上,这样就会造成hash冲突的概率变大.<br>
所以,算hash值的时候,先用高16位与低16位取抑或,这样就能降低hash冲突的概率了.

#### hashMap的容量为什么必须是2的n次幂?
这是因为扩容时,会重新计算所有旧的Node的位置.<br>
假如旧的容量是`a`,那么新的容量就是`2a`<br>
原来在`j`位置上的node新位置只有两种情况:
`j`或者`a+j`<br>
因为旧的那些结点模`a`等于`j`,那么模`2a`必然等于`j`或者`a+j`,这样扩容时,同一位置的node就可以分成两批移动了,提高扩容效率.<br>
而且计算新位置的这里的算法特别巧妙:<br>
假设原来的大小是16,有两个元素在一个位置:2和18,它们和`16-1``&`运算<br>
`0000 0010` 和`0001 0010`<br>
`0000 1111` --`0000 1111`<br>
要判断他们的新位置,不需要和`32-1`进行`&`运算,只需要和`32`进行`&`运算看高一位是0还是1就行了,这样很方便的就能将他们均匀的分散开.
`0000 0010` 和`0001 0010` <br>
`0001 0000` --`0001 0000` <br>
源码:
```
                        do {
                            next = e.next;
                            //注意,这里没有和(oldGap-1)进行`&`运算 只有等于0和oldCap两种结果
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
```

#### 默认负载因子为什么是0.75
1. 假如负载因子是1,那么数组满了才会扩容,但很多情况下,数组满了很大概率会存在hash冲突,,hash值算出的索引分布不一定是分布在数组的每一个位置,有了hash冲突之后,就会形成链表或者红黑树,这样查询效率就降低了.
2. 假如负载因子是0.5,那么数组空间用到了一半就得扩容,然后就得重新计算所有node的新位置,虽然查询变快了,但是空间利用率降低了.<br>

所以官方用了综合表现比较好的0.75.

#### hashMap不是线程安全的
在1.7中,hash冲突时,链表插入采用的时头插法,比如多线程对容量为2的数组依次put 5,3,7会产生7-》3-》5这种结构,然后扩容成4,将node加到新位置,有可能造成3—》7-》3的循环.

在1.8中,两个线程分别put 3和7
```
   if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```
如果hash计算出的索引位置上没有node的话,就有可能造成数据覆盖,丢失了一部分数据.还有`resize`时插入删除等.


#### 为什么树化的阈值是8?
根据官方注释:
```
Ideally, under random hashCodes, the frequency of
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
理想情况下,容器中的结点符合泊松分布,链表长度大于8的情况特别小,所以选了8来做阈值.而且,当链表长度为大于8时,链表的平均查询时间大于`8/2=4`,比红黑树的平均查询时间`lg8=3`要大,这时候转成红黑树就能提高查询效率了.
而且,TreeNode需要记录praent、left、right等值,比Node占空间,所以小于6的时候,转成链表比较好.

#### hashMap和HashTable的区别
1. hashTable的初始大小为11,每次扩容2n+1
2. hashTable里的大部分方法都用`synchronized`修饰,是线程安全的
3. hashTable不允许key是null,hashMap允许.
4. hashTable是数组加链表存数据,没有红黑树
5. 单线程下hashMap效率高,hashTable基本被淘汰了,为了线程安全可以用ConcurrentHashMap


#### LinkedHashMap
LinkedHashMap继承自HashMap,所以底层存储结构差不多,但比它多了一条双链表保证有序性

#### hashSet
`hashSet`实际上也是借用`hashMap`来存值的.
```
private transient HashMap<E,Object> map;

public HashSet() {
        map = new HashMap<>();
    }
```

