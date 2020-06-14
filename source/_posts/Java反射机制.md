---
title: Java反射机制
categories: 
 - 技术
tags:
 - Java
---

反射是Java在运行时,可以访问、检测和修改它的本身状态或行为的一种能力.

首先的知道Class的概念: JVM为每种类型的类维护一个Class对象,里面包含了该类的结构、属性和方法等信息.我们使用javac编译,产生的.class文件保存着该类的对象,JVM加载一个类就是指加载该.class文件.

关于反射,以我目前的经验来看,可以主要关注下面几点:
1. Field: 描述该类的属性;
2. Method: 该类的方法
3. Constructor: 该类的构造器
4. Modifier: 修饰符信息

先准备两个类用于实验:
```
public class Person {
    public String name; // 姓名 公有
    protected String age;   // 年龄 保护
    private String hobby;   // 爱好   私有

    public Person(String name, String age, String hobby) {
        this.name = name;
        this.age = age;
        this.hobby = hobby;
    }
    public String getHobby() {
        return hobby;
    }
}

public class Employee extends Person {
    public static Integer totalNum = 0; // 员工数
    public int empNo;   // 员工编号 公有
    protected String position;  // 职位 保护
    private int salary; // 工资   私有

    public void sayHello() {
        System.out.println(String.format("Hello, 我是 %s, 今年 %s 岁, 爱好是%s, 我目前的工作是%s, 月入%s元\n", name, age, getHobby(), position, salary));
    }
    private void work() {
        System.out.println(String.format("My name is %s, 工作中勿扰.", name));
    }
    public Employee(String name, String age, String hobby, int empNo, String position, int salary) {
        super(name, age, hobby);
        this.empNo = empNo;
        this.position = position;
        this.salary = salary;
        Employee.totalNum++;
    }
}
```

##### 获取class对象
获取class对象的方式有三种: 
1. 通过Class类的forName方法,通过路径获取对象;
2. 直接通过类名.class方法,获取该类的class对象;
3. 通过该类的一个实例的.getClass()方法.
```
public class ClassTest {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Class c1 = Class.forName("reflect.Employee");   // 第1种，forName 方式获取Class对象
        Class c2 = Employee.class;      // 第2种，直接通过类获取Class对象
        Employee employee = new Employee("小明", "18", "写代码", 1, "Java攻城狮", 100000);
        Class c3 = employee.getClass();    // 第3种，通过调用对象的getClass()方法获取Class对象

        if (c1 == c2 && c1 == c3) {     // 可以通过 == 比较Class对象是否为同一个对象
            System.out.println("c1、c2、c3 为同一个对象");
            System.out.println(c1);     // class reflect.Employee
        }
    }
}
```

##### 通过反射来创建实例
通过反射来创建一个类的实例的方法有两种:
1. 使用Class对象的newInstance()方法来创建Class对象对应类的实例
2. 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例
```
public class NewInstanceTest {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class c = Date.class;
        Date date1 = (Date) c.newInstance();    // 第1种方式：使用Class对象的newInstance()方法来创建Class对象对应类的实例
        System.out.println(date1);      // Wed Dec 19 22:57:16 CST 2018

        long timestamp =date1.getTime();
        Constructor constructor = c.getConstructor(long.class); 
        Date date2 = (Date)constructor.newInstance(timestamp);  // 第2种方式：先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例
        System.out.println(date2);  // Wed Dec 19 22:57:16 CST 2018
    }
}
```

#### 获取类信息的部分API

`String getName()` 获取这个Class的类名

`Constructor[] getDeclaredConstructors()` 返回这个类的所有构造器的对象数组，包含保护和私有的构造器；相近的方法 getConstructors() 则返回这个类的所有公有构造器的对象数组，不包含保护和私有的构造器

`Method[] getDeclaredMethods()` 返回这个类或接口的所有方法，包括保护和私有的方法，不包括超类的方法；相近的方法 getMethods() 则返回这个类及其超类的公有方法的对象数组，不含保护和私有的方法

`Field[] getDeclaredFields()` 返回这个类的所有域的对象数组，包括保护域和私有域，不包括超类的域；还有一个相近的API getFields()，返回这个类及其超类的公有域的对象数组，不含保护域和私有域

`int getModifiers()` 返回一个用于描述Field、Method和Constructor的修饰符的整形数值，该数值代表的含义可通过Modifier这个类分析

`Modifier` 类 它提供了有关Field、Method和Constructor等的访问修饰符的信息，主要的方法有：toString(int modifiers)返回整形数值modifiers代表的修饰符的字符串；isAbstract是否被abstract修饰；isVolatile是否被volatile修饰；isPrivate是否为private；isProtected是否为protected；isPublic是否为public；isStatic是否为static修饰；等等，见名知义

#### 运行时查看对象数据域实际内容的相关API
`Class<?> getComponentType()` 返回数组类里组件类型的 Class，如果不是数组类则返回null

`boolean isArray()` 返回这个类是否为数组，同类型的API还有 isAnnotation、isAsciiDigit、isEnum、isInstance、isInterface、isLocalClass、isPrimitive 等

`int Array.getLength(obj)` 返回数组对象obj的长度

`Object Array.get(obj, i)` 获取数组对象下标为i的元素

`boolean isPrimitive()` 返回这个类是否为8种基本类型之一，即是否为boolean, byte, char, short, int, long, float, 和double 等原始类型

`Field getField(String name)` 获取指定名称的域对象

`AccessibleObject.setAccessible(fields, true)` 当访问 Field、Method 和 Constructor 的时候Java会执行访问检查，如果访问者没有权限将抛出SecurityException，譬如访问者是无法访问private修饰的域的。通过设置 setAccessible(true) 可以取消Java的执行访问检查，这样访问者就获得了指定 Field、Method 或 Constructor 访问权限

`Class<?> Field.getType()` 返回一个Class 对象，它标识了此 Field 对象所表示字段的声明类型

`Object Field.get(Object obj)` 获取obj对象上当前域对象表示的属性的实际值，获取到的是一个Object对象，实际使用中还需要转换成实际的类型，或者可以通过 getByte()、getChar、getInt() 等直接获取具体类型的值

`void Field.set(Object obj, Object value)` 设置obj对象上当前域表示的属性的实际值

#### 调用任意方法
上面查看Field的实际值是通过 Field 类的 get() 方法，与之类似，Method 调用方法是通过 Method 类的 invoke 方法

##### 调用任意方法相关的API
`Method getMethod(String name, Class<?>... parameterTypes)` 获取指定的 Method，参数 name 为要获取的方法名，parameterTypes 为指定方法的参数的 Class，由于可能存在多个同名的重载方法，所以只有提供正确的 parameterTypes 才能准确的获取到指定的 Method

`Object invoke(Object obj, Object... args)` 执行方法，第一个参数执行该方法的对象，如果是static修饰的类方法，则传null即可；后面是传给该方法执行的具体的参数值

```
public class ClassTest {

    public static void main(String[] args)
            throws ClassNotFoundException, IllegalAccessException, InstantiationException,
            NoSuchFieldException, NoSuchMethodException, InvocationTargetException {
        Class c1 = Class.forName("reflection.Employee"); // 第1种，forName 方式获取Class对象
        Class c2 = Employee.class; // 第2种，直接通过类获取Class对象
        Employee employee = new Employee("小明", "18", "写代码", 1, "Java攻城狮", 100000);
        Class c3 = employee.getClass(); // 第3种，通过调用对象的getClass()方法获取Class对象

        if (c1 == c2 && c1 == c3) { // 可以通过 == 比较Class对象是否为同一个对象
            System.out.println("c1、c2、c3 为同一个对象");
            System.out.println(c1); // class reflect.Employee
        }

        //获取属性
        System.out.println(c1.getDeclaredField("empNo").get(employee));

        Method method = c2.getDeclaredMethod("sayHello");
        method.setAccessible(true);
        //调用方法
        System.out.println(method.invoke(employee));
    }
}
```


#### 反射的优点：

- 可扩展性 ：
应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。

- 类浏览器和可视化开发环境：
一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。

- 调试器和测试工具 ： 调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。

#### 反射的缺点：
尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

- 性能开销 ：
反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。

- 安全限制 ：
使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。

- 内部暴露 ：
由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

参考来源:https://mp.weixin.qq.com/s?__biz=MzI1NDU0MTE1NA==&mid=2247483785&idx=1&sn=f696c8c49cb7ecce9818247683482a1c&chksm=e9c2ed84deb564925172b2dd78d307d4dc345fa313d3e44f01e84fa22ac5561b37aec5cbd5b4&scene=0#rd
