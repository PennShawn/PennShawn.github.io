---
title: 《Effective Java》 Item 10.Obey the general contract when overriding equals
categories :
- 技术
tags :
- Java
- Effective Java
---

Overrinding the equals method seems simple,but there are many ways to get it wrong,and consequences can be dire. The easiest way to avoid problem is not to override the equals method, in which case each instance of the class is equal only to itself.


when you override the equals method, you must adhere to its general contract. Here is contract, from the specification of Object:

-Reflexive
-Symmetric
-Transitive
-Consistent
-For any non-null refrence value x, `x.equals(null)` must return false. 

```
package com.zejian.test;
import java.util.ArrayList;
import java.util.List;
/** * 反面例子 * @author zejian */
public class AbnormalResult {
	public static void main(String[] args) {
		List<A> list = new ArrayList<A>();
		A a = new A();
		B b = new B();
		list.add(a);
		System.out.println("list.contains(a)->" + list.contains(a));
		System.out.println("list.contains(b)->" + list.contains(b));
		list.clear();
		list.add(b);
		System.out.println("list.contains(a)->" + list.contains(a));
		System.out.println("list.contains(b)->" + list.contains(b));
	}
	static class A {
		@Override
		public boolean equals(Object obj) {
			return obj instanceof A;
		}
	}
	static class B extends A {
		@Override
		public boolean equals(Object obj) {
			return obj instanceof B;
		}
	}
}
```
输出:
```
list.contains(a)->true

list.contains(b)->false

list.contains(a)->true

list.contains(b)->true
```
list.contains()最终调用的是equals方法,代码list.contains(a)，实际上调用了a.equal(a[i]),其中a[i]是集合中的元素而且为B类型(只添加了b对象)，由于B类型肯定是A类型（B继承了A），所以a[i] instanceof A,结果为true


```
import java.util.HashMap;
import java.util.Map;
public class MapTest {
	public static void main(String[] args) {
		Map<String,Value> map1 = new HashMap<String,Value>();
		String s1 = new String("key");
		String s2 = new String("key");	
		Value value = new Value(2);
		map1.put(s1, value);
		System.out.println("s1.equals(s2):"+s1.equals(s2));
		System.out.println("map1.get(s1):"+map1.get(s1));
		System.out.println("map1.get(s2):"+map1.get(s2));
		
		
		Map<Key,Value> map2 = new HashMap<Key,Value>();
		Key k1 = new Key("A");
		Key k2 = new Key("A");
		map2.put(k1, value);
		System.out.println("k1.equals(k2):"+s1.equals(s2));
		System.out.println("map2.get(k1):"+map2.get(k1));
		System.out.println("map2.get(k2):"+map2.get(k2));
	}
	
	/**
	 * 键
	 * @author zejian
	 *
	 */
	static class Key{
		private String k;
		public Key(String key){
			this.k=key;
		}
		
		@Override
		public boolean equals(Object obj) {
			if(obj instanceof Key){
				Key key=(Key)obj;
				return k.equals(key.k);
			}
			return false;
		}
	}
	
	/**
	 * 值
	 * @author zejian
	 *
	 */
	static class Value{
		private int v;
		
		public Value(int v){
			this.v=v;
		}
		
		@Override
		public String toString() {
			return "类Value的值－－>"+v;
		}
	}
}
```
结果:
```

s1.equals(s2):true
map1.get(s1):类Value的值－－>2
map1.get(s2):类Value的值－－>2
k1.equals(k2):true
map2.get(k1):类Value的值－－>2
map2.get(k2):null
```


在重写equals() 方法时，一般都是推荐使用getClass来进行类型判断（除非所有的子类有统一的语义才使用instanceof),防止出现对于不同的子类e1,e2, 父类p即等于e1又等于e2.

参考:https://blog.csdn.net/javazejian/article/details/51348320#

