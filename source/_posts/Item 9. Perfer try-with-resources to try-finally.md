---
title: 《Effective Java》Item 9. Perfer try-with-resources to try-finally
categories :
- 技术
tags :
- Java
- Effective Java
---

Always use try-with-resources in preference to try-finally when working with resources that must be closed.

```
try(InputStream in =new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)){

    byte[] buf = new byte[BUFFER_SIZE]
    ...    

}
```

对于一个资源文件`Connection`
```
public class Connection implements AutoCloseable {

    public void sendData() {
        System.out.println("sendData");
    }

    @Override
    public void close() throws Exception {
        System.out.println("close");
    }
}
```
##### 传统的打开关闭方式
```
public class TryCacheFinally {
    public static void main(String[] args) {
        Connection connection = null;
        try {
            connection = new Connection();
            connection.sendData();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                connection.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
要保证关闭写起来比较繁琐,而且当`sendData`和`close`方法都抛异常时,`sendData`的异常会被吞掉,只输出`close`的异常,造成异常丢失.
##### try-with-source 
```
    public static void main(String[] args) {
        try (Connection connection = new Connection()) {
            connection.sendData();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
不仅代码优雅很多,而且发生异常时,也能打印所有异常.
原理是编译器在编译时,自动帮我们优化了代码.
```

public class TryWithSource {
    public TryWithSource() {
    }

    public static void main(String[] var0) {
        try {
            Connection var1 = new Connection();
            Throwable var2 = null;

            try {
                var1.sendData();
            } catch (Throwable var12) {
                var2 = var12;
                throw var12;
            } finally {
                if (var1 != null) {
                    if (var2 != null) {
                        try {
                            var1.close();
                        } catch (Throwable var11) {
                            var2.addSuppressed(var11);
                        }
                    } else {
                        var1.close();
                    }
                }

            }
        } catch (Exception var14) {
            var14.printStackTrace();
        }

    }
}
```
不但加上了finally,而且有多个异常时,` var2.addSuppressed(var11);`将一个异常附加到了另一个异常上面,因此不会丢失异常.
##### 注意!
在try-with-resource时,要单独声明最底层的资源,保证对应的clise方法一定会被调用.例:
```
try (FileInputStream fin = new FileInputStream(new File("input.txt"));
                GZIPOutputStream out = new GZIPOutputStream(new FileOutputStream(new File("out.txt")))) {
            byte[] buffer = new byte[4096];
            int read;
            while ((read = fin.read(buffer)) != -1) {
                out.write(buffer, 0, read);
            }
        }
        catch (IOException e) {
            e.printStackTrace();
        }
```
发生异常时,最终会调用`GZIPOutputStream`类的`close`方法
```
public void close() throws IOException {
    if (!closed) {
        finish();
        if (usesDefaultDeflater)
            def.end();
        out.close();
        closed = true;
    }
}
```
当`finish`方法发生异常时,`out.close`方法就不会被调用,资源没有成功被关闭.
所以,要单独声明`FileOutputStream`.
```
public class TryWithResource {
    public static void main(String[] args) {
        try (FileInputStream fin = new FileInputStream(new File("input.txt"));
                FileOutputStream fout = new FileOutputStream(new File("out.txt"));
                GZIPOutputStream out = new GZIPOutputStream(fout)) {
            byte[] buffer = new byte[4096];
            int read;
            while ((read = fin.read(buffer)) != -1) {
                out.write(buffer, 0, read);
            }
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
