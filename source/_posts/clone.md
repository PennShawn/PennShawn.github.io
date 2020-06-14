---
title: clone
categories: 
 - 技术
tags:
 - Java
---

clone是Object类中的方法
```
protected native Object clone() throws CloneNotSupportedException;
```
既然是protected,那么在别的地方要clone一个类的话,就要重写一下clone方法.

但是光这样是不行的,直接调用程序会抛`CloneNotSupportedException`异常,一个类被clone必须实现Cloneable接口,这个接口没有任何方法,只是像`Serializable`一样作为一个可以clone的标识.


##### clone的过程
我们知道,对象是放在堆内存中的,clone一个对象,会先在堆内存中申请同样大小的空间,然后将原对像的值填充过去,一个新的对象就产生了,然后将这个对象的引用返回,之后就可以对这个克隆对象进行操作了.

##### 深克隆、浅克隆
上面讲的就是浅克隆的过程,如果一个克隆对象内部有引用类型时,填充的只是源对象的该属性的引用.如:
```
class Head {

    int high;

    public Head(int high) {
        this.high = high;
    }

    public int getHigh() {
        return high;
    }

    public void setHigh(int high) {
        this.high = high;
    }
}

public class Person implements Cloneable {

    private static final long serialVersionUID = 120904406677219163L;

    Head head;

    public Person() {
        head = new Head(2);
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        Person p1 = new Person();
        Person p2 = (Person) p1.clone();
        p1.head.setHigh(1);
        System.out.println(p2.head.high); // 输出 1
    }
}

```
我们将p1克隆出p2后,改变p1中head里的属性,p2中head里的属性也变了,因为p1和p2的head指向的都是同一个对象.

如果我们不希望这样就得采用深克隆了.

除了让head实现cloneable外,person的clone方法不能简单的返回super.clone();了
```
  @Override
    public Object clone() throws CloneNotSupportedException {
        Person p = (Person) super.clone();
        p.head = (Head) head.clone();
        return p;
    }
```
