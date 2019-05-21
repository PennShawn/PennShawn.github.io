---
title:  List.contains()方法性能
categories :
- 技术
tags :
- Java
---


无论是ArrayList 还是 LinkedList,contains()方法都是返回所查元素的索引,需要遍历List,时间复杂度O(n),比较耗时.
源码:
```
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
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
    //LinkedList
   public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```
数据量大并且频繁调用时最好用Set
hsahSet用`private transient HashMap<E,Object> map;`
保存数据,
它的add()方法:
```
    private static final Object PRESENT = new Object();
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

contains()方法:
```
 public boolean contains(Object o) {
    return map.containsKey(o);
 }
```



