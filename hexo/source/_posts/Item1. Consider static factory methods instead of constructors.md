---
title: 《Effective Java》Item1. Consider static factory methods instead of constructors
categories :
- 技术
tags :
- Java
- Effective Java
---

### Advantage 1. name
　静态工厂方法可以通过方法的名字很直接的传递方法很多信息,而不像传统的构造方法一样名字只能和类名相同,用户只能通过传递的参数和注释文档来判断调用哪个方法.一个很好的例子:

有一个RandomIntGenerator 类，产生随机的int类型的整数。如下所示： 
```
public class RandomIntGenerator {
private final int min;
private final int max;

public int next() {...}
}
```
这个生成器接收最大值和最小值两个参数并且生成介于两者之间的随机数。注意到两个属性min和max被final修饰，所以必须初始化它们。可以在定义它们时就初始化或者通过构造器来初始化。通过构造器初始如下：
```
public RandomIntGenerator(int min, int max) {
    this.min = min;
    this.max = max;
}
```
现在，我们要提供给这样一个功能，客户端仅设置一个最小值然后产生一个介于此值和Integer.MAX_VALUE之间的整数。所以我们添加了第二个构造器：
```
public RandomIntGenerator(int min) {
    this.min = min;
    this.max = Integer.MAX_VALUE;
}
```
到目前为止，一切正常。但是，我们同样要提供一个构造器设置一个最大值，然后产生一个介于Integer.MIN_VALUE和最大值的整数。我们添加第三个构造器如下：
```
public RandomIntGenerator(int max) {
    this.min = Integer.MIN_VALUE;
    this.max = max;
}
```
如果你这样做了，将会出现编译错误：Duplicate method RandomIntGenerator(int) in type RandomIntGenerator。那里出错了？毫无疑问，问题在于构造器没有名字。因此，一个类仅有一个特定方法签名的构造器。同样的，你不能定义方法签名相同的（返回值、名称、参数类型及个数均相同）两个方法。这就是为什么当我们试着添加构造器RandomIntGenerator(int max) 时，会得到上述的编译错误，原因是我们已经有一个构造器 RandomIntGenerator(int min)。

像这样的情况我们该如何处理呢？幸好有其他方式可以使用：静态工厂方法，通过使用简单的公共静态方法返回一个类的实例。你可能在无意识中已经使用过这种技术。你有没有写过Boolean.valueOf?，就像下面这样：
```
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```
将静态工厂应用到RandomIntGenerator类，得到
```
 public class RandomIntGenerator {

    private final int min;
    private final int max;

    private RandomIntGenerator(int min, int max) {
        this.min = min;
        this.max = max;
    }

    public static RandomIntGenerator between(int max, int min) {
        return new RandomIntGenerator(min, max);
    }

    public static RandomIntGenerator biggerThan(int min) {
        return new RandomIntGenerator(min, Integer.MAX_VALUE);
    }

    public static RandomIntGenerator smallerThan(int max) {
        return new RandomIntGenerator(Integer.MIN_VALUE, max);
    }

    public int next() {...}
}
```
注意到构造器被private修饰确保类仅能通过静态工厂方法来初始化。并且当你使用RandomIntGenerator.between(10,20)而不是new RandomIntGenerator(10,20)来产生整数时你的意图被清晰的表达了。值得注意的是，这个技术和 Gang of Four的工厂设计模式不同。此外，任何类可以提供静态工厂方法替代构造器。那么此种技术的优点和缺点是什么呢？我们已经提到过静态工厂方法的第一个优点：静态工厂方法拥有名字。这有两个直接的好处:
1.我们可以给静态方法提供一个有意义的名字
2.我们可以给静态方法提供参数类型、参数个数相同的几个构造器，在传统构造器中是不能这样做的.

 

### Advantage 2.new object
　　静态工厂方法在调用的时候不需要每次都创建一个新的对象.一个很好的例子是 java.lang.Boolean中的valueOf方法
```
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```
多次调用也不会重复创建对象
### Advantage 3.return any subtype of their return type(范型)
　　当我们不仅需要返回随机的int值时,我们可以在之前提到的RandomGenerators中加上静态工厂方法
```
public final class RandomGenerators {
    // Suppresses default constructor, ensuring non-instantiability.
    private RandomGenerators() {}

    public static final RandomGenerator<Integer> getIntGenerator() {
        return new RandomIntGenerator(Integer.MIN_VALUE, Integer.MAX_VALUE);
    }

    public static final RandomGenerator<String> getStringGenerator() {
        return new RandomStringGenerator('');
    }
}
```
这样需要返回随机字符串时,直接调用RandomGenerator.getStringGenerator().get....
另一个例子就是现在经常用到的com.google.common.collect中的Maps.newHaskMap()等.(guava)
Advantage 4.返回对象的类型可根据传入的参数变化?
　　同样收集new一个HashMap.传统方式

Map<String, Integer> map = new HashMap<String, Integer>();
 而用了静态工厂方法
public static <K, V> HashMap<K, V> newHashMap() {
  return new HashMap<K, V>();
}
则可以直接 
Map<String, Integer> map =Maps.newHaskMap()
Advantage 5.延迟加载?
如JDBC中的

DriverManager.getConnection()
调用的时候connection不需要已经建立.
 

### Disadvantage
1.如果我们在类中将构造函数设为private，只提供静态工厂方法来构建对象，那么我们将不能通过继承扩展该类。 
但是这也会鼓励我们使用复合而不是继承来扩展类。
2.构建对象的静态工厂方法并没有像构造器那样明确标识出来，不能和其他静态方法很方便地区分开来。 
如果类中只提供静态工厂方法而不是构造器，要想查明如何实例化一个类将会变得困难。 
我们可以通过遵循静态工厂方法的命名规范来弥补这一劣势：
```
valueOf - 返回的实例与它的参数具有相同的值，一般作为类型转换使用，例如Boolean.valueOf(boolean)
of - valueOf的更为简洁的替代。
getInstance - 返回的实例通过方法的参数来描述，但不能说与参数具有同样的值。对于Singleton来说，使用无参getInstance，返回唯一的实例。
newInstance - 像getInstance一样，但其能够确保每次都返回新的对象。
getType - 像getInstance一样，但此方法返回的对象是另一个不同的类。
newType - 像getType一样，但每次返回一个新对象。
```
引用链接:http://honoka.cnblogs.com。https://yq.aliyun.com/articles/17013.




