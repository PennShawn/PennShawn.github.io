---
title: Java的三种代理模式——静态代理、JDK动态代理和cglib代理
categories: 
 - 技术
tags:
 - Java
---

代理模式是一种设计模式，提供了对目标对象额外的访问方式，即通过代理对象访问目标对象，这样可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能。

简言之，代理模式就是设置一个中间代理来控制访问原目标对象，以达到增强原对象的功能和简化访问方式。

#### 静态代理
首先是静态代理,就是逻辑最简单的一种代理模式,如果A类继承B类,但是A类中的一个方法需要扩展,又不想改A类,那么就可以用静态代理.<br>
新建一个代理类C类取实现B类,但是C类的构造函数就要传入一个B类对象,如果把一个A类的实例传进去,那么C类就有了A类的所有功能,在调用需要用到的方法的时候,就可以调用A类的这个方法,并且加一些扩展,但是在外界看起来只是调用C类的这个方法.
```
//接口
public interface IUserDao {
    //需要代理的方法
    public void save();
}

//实现接口的A类
public class UserDao implements IUserDao {

    @Override
    public void save() {
        System.out.println("save");
    }
}

//C类 构造方法里就获取了A类的实例
public class UserDaoProxy implements IUserDao {

    IUserDao target;

    public UserDaoProxy(IUserDao target) {
        this.target = target;
    }

    
    @Override
    //对A类的save()方法进行扩展
    public void save() {
        System.out.println("before");
        target.save();
        System.out.println("after");
    }
}

//使用者
public class TestStaticProxy {

    public static void main(String[] args) {
        IUserDao target = new UserDao();
        UserDaoProxy proxy = new UserDaoProxy(target);
        //使用者看起来还是只调用了save()方法,但实际上被代理类扩展了逻辑
        proxy.save();
    }
}

```


#### 动态代理
动态代理利用了JDK API,在运行期间动态地在内存中构建代理对象,从而实现对目标对象的代理功能。动态代理又被称为JDK代理或接口代理。

- 静态代理与动态代理的区别主要在：
静态代理在编译时就已经实现，编译完成后代理类是一个实际的class文件
动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中

**动态代理对象不需要实现接口，但是要求目标对象必须实现接口，否则不能使用动态代理。**

JDK中生成代理对象主要涉及的类有
```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
```
主要的方法:

Proxy:
```
//返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序
public static Object newProxyInstance(ClassLoader loader,//指定当前目标对象使用类加载器
Class<?>[] interfaces,//目标对象实现的接口的类型
InvocationHandler h //事件处理器
)
```
InvocationHandler:
```
 Object    invoke(Object proxy, Method method, Object[] args) 
// 在代理实例上处理方法调用并返回结果。
```

示例代码:
```
//接口
public interface IUserDao {
    public void save();
}

//目标对象
public class UserDao implements IUserDao {
    @Override
    public void save() {
        System.out.println("save");
    }
}

//代理类
public class ProxyFactory {

    //维护一个目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), new InvocationHandler() {

                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args)
                            throws Throwable {
                        System.out.println("start");
                        Object returnValue = method.invoke(target, args);
                        System.out.println("end");
                        return returnValue;
                    }
                });
    }
}

//调用者
public class TestProxy {

    public static void main(String[] args) {

        //        IUserDao target = new UserDao();
        //        UserDaoProxy proxy = new UserDaoProxy(target);
        //        proxy.save();

        IUserDao target = new UserDao();
        IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
        proxy.save();
    }
}
```

##### 为什么动态代理的类要实现接口
因为我们最后创建的代理对象首先是继承了proxy类的,我们还需要获取代理对象的方法,而Java只能单继承,这时候就得通过接口来实现了.


参考来源:https://segmentfault.com/a/1190000011291179<br>
扩展: https://juejin.im/post/5c1ca8df6fb9a049b347f55c
