---
title: CopyOnWriteArrayList
categories: 
 - 技术
tags:
 - Java
 - 数据结构
 - 源码
---


普通的vector来实现线程安全的ArrayList时,方法都是由`synchronized`修饰的,读和读,读和写,写和写都会造成锁竞争.但其实只有写的时候才会改变数据,造成并发问题.所以可以用一种性能更好的数据结构,`CopyOnWriteArrayList`.<br>
顾名思义,CopyOnWriteArrayList会在写的时候复制一份原有的数据进行操作,这个操作会加锁,所以同时只会有一个线程进行写操作.
```
 /**
     * Replaces the element at the specified position in this list with the
     * specified element.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);

            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
    
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
    ...
```
在读的时候不加锁
```
public E get(int index) {
        return get(getArray(), index);
    }

 @SuppressWarnings("unchecked")
    private E get(Object[] a, int index) {
        return (E) a[index];
    }

```
缺点:
1. 内存占用问题,因为写的时候会复制一份相同的数据,所以会占用一些额外的内存
2. 不是强一致的,因为写的时候是复制原数据操作,操作完了再将引用指向新数据,所以数据不是实时一致的,只能保证最终一致性.
