---
title: ConcurrentHashMap
categories: 
 - 技术
tags:
 - Java
 - 数据结构
 - 源码
---

HashMap的源码已经分析过了,但是我们知道,HashMap不是线程安全的,如果要在有着并发操作的场景里,就需要用到ConcurrentHashMap了.
ConcurrentHashMap在JDK1.8中的实现已经抛弃了Segment分段锁机制，利用CAS+Synchronized来保证并发更新的安全，底层采用数组+链表+红黑树的存储结构,这点和HashMap类似.
```
    /**
     * The array of bins. Lazily initialized upon first insertion.
     * Size is always a power of two. Accessed directly by iterators.
     */
    transient volatile Node<K,V>[] table;

    /**
     * The next table to use; non-null only while resizing.
     */
    private transient volatile Node<K,V>[] nextTable;

    /**
     * Base counter value, used mainly when there is no contention,
     * but also as a fallback during table initialization
     * races. Updated via CAS.
     */
    private transient volatile long baseCount;

    /**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
    private transient volatile int sizeCtl;

    /**
     * The next table index (plus one) to split while resizing.
     */
    private transient volatile int transferIndex;

    /**
     * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
     */
    private transient volatile int cellsBusy;

    /**
     * Table of counter cells. When non-null, size is a power of 2.
     */
    private transient volatile CounterCell[] counterCells;

    // views
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;
```
ConcurrentHashMap用`table`存储数据,`sizeCtl`默认为0,在table初始化后,默认时table大小的0.75倍`(n - (n >>> 2))`,用Node<K,V>来保存key,value和hash值.
```
class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
    ... 
}
```
这次主要是分析put操作.
```
for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
               ...
```
这里可以看到if判断可能会有4种情况:

##### 1.table未初始化,那么会首先初始化

```
 if (tab == null || (n = tab.length) == 0)
                tab = initTable();
```
```
  /**
     * Initializes table, using the size recorded in sizeCtl.
     */
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```
sizeCtl默认值是0,如果ConcurrentHashMap实例化时有传参数，sizeCtl会是一个2的幂次方的值.当一个线程执行table初始化时,该线程会执行Unsafe.compareAndSwapInt方法修改sizeCtl为-1.如果一个线程发现`sizeCtl<0`说明有别的线程正在初始化table,那么当前线程需要让出时间片,防止多个线程put时table被多次初始化.<br>

##### 2.新的元素映射的位置没有旧元素
```
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
```
这种情况会使用`casTabAt`将新元素插入到table[i]的头结点上,如果没有插入成功,说明有其他线程已经对table[i]的位置进行了修改,则不执行操作再走一遍循环,继续判断一遍.

##### 3.table[i]的hash值为-1
```
 else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
```
这种情况说明有其他线程在执行扩容操作,帮助他们一起扩容,提高性能.

##### 4.table[i]位置有元素了
```
else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
```
这里就得用synchorized锁住f可就是table[i]位置的第一个元素,防止同时有多个线程对同一个位置上的数据进行操作,也可以分为两种情况<br>
1.原有位置是个链表,判断是否需要插入新结点,如果需要,那么就会遍历这个链表,将新结点插到链表尾部,并且几下链表长度,最后判断是否需要转换成红黑树.
```
 if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
```
2.原有位置的结点是个树的结点,则判断是否需要将新结点插入到红黑树中.

#### 扩容
将新的结点加上后,会调用`addCount(1L, binCount);`,判断是否需要进行扩容操作.

