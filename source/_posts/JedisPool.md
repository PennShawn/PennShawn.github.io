---
title: JedisPool
categories: 
 - 技术
tags:
 - Java
 - redis
 - 源码
---


当向JedisPool请求jedis时,会调用borrowObject方法
```
 public T borrowObject(long borrowMaxWaitMillis) throws Exception {
        assertOpen();

        AbandonedConfig ac = this.abandonedConfig;
        if (ac != null && ac.getRemoveAbandonedOnBorrow() &&
                (getNumIdle() < 2) &&
                (getNumActive() > getMaxTotal() - 3) ) {
            removeAbandoned(ac);
        }

        PooledObject<T> p = null;

        // Get local copy of current config so it is consistent for entire
        // method execution
        boolean blockWhenExhausted = getBlockWhenExhausted();

        boolean create;
        long waitTime = System.currentTimeMillis();

        while (p == null) {
            create = false;
            if (blockWhenExhausted) {
                p = idleObjects.pollFirst();
                if (p == null) {
                    p = create();
                    if (p != null) {
                        create = true;
                    }
                }
                if (p == null) {
                    if (borrowMaxWaitMillis < 0) {
                        p = idleObjects.takeFirst();
                    } else {
                        p = idleObjects.pollFirst(borrowMaxWaitMillis,
                                TimeUnit.MILLISECONDS);
                    }
                }
                if (p == null) {
                    throw new NoSuchElementException(
                            "Timeout waiting for idle object");
                }
                if (!p.allocate()) {
                    p = null;
                }
            } else {
                p = idleObjects.pollFirst();
                if (p == null) {
                    p = create();
                    if (p != null) {
                        create = true;
                    }
                }
                if (p == null) {
                    throw new NoSuchElementException("Pool exhausted");
                }
                if (!p.allocate()) {
                    p = null;
                }
            }

            if (p != null) {
                try {
                    factory.activateObject(p);
                } catch (Exception e) {
                    try {
                        destroy(p);
                    } catch (Exception e1) {
                        // Ignore - activation failure is more important
                    }
                    p = null;
                    if (create) {
                        NoSuchElementException nsee = new NoSuchElementException(
                                "Unable to activate object");
                        nsee.initCause(e);
                        throw nsee;
                    }
                }
                if (p != null && (getTestOnBorrow() || create && getTestOnCreate())) {
                    boolean validate = false;
                    Throwable validationThrowable = null;
                    try {
                        validate = factory.validateObject(p);
                    } catch (Throwable t) {
                        PoolUtils.checkRethrow(t);
                        validationThrowable = t;
                    }
                    if (!validate) {
                        try {
                            destroy(p);
                            destroyedByBorrowValidationCount.incrementAndGet();
                        } catch (Exception e) {
                            // Ignore - validation failure is more important
                        }
                        p = null;
                        if (create) {
                            NoSuchElementException nsee = new NoSuchElementException(
                                    "Unable to validate object");
                            nsee.initCause(validationThrowable);
                            throw nsee;
                        }
                    }
                }
            }
        }

        updateStatsBorrow(p, System.currentTimeMillis() - waitTime);

        return p.getObject();
    }
```
##### 创建新的jedis连接
当没有jedis连接空闲的时候,会创建新的jedis连接
```
 private PooledObject<T> create() throws Exception {
        int localMaxTotal = getMaxTotal();
        long newCreateCount = createCount.incrementAndGet();
        if (localMaxTotal > -1 && newCreateCount > localMaxTotal ||
                newCreateCount > Integer.MAX_VALUE) {
            createCount.decrementAndGet();
            return null;
        }

        final PooledObject<T> p;
        try {
            p = factory.makeObject();
        } catch (Exception e) {
            createCount.decrementAndGet();
            throw e;
        }

        AbandonedConfig ac = this.abandonedConfig;
        if (ac != null && ac.getLogAbandoned()) {
            p.setLogAbandoned(true);
        }

        createdCount.incrementAndGet();
        allObjects.put(new IdentityWrapper<T>(p.getObject()), p);
        return p;
    }

```
其中,通过createCount这个AtomicLong来控制并发情况下的jedis连接数不会溢出.
##### 返回空闲连接
通过LinkedBlockingDeque<PooledObject<T>> idleObjects来控制多线程情况下的jedis获取.
```
 p = idleObjects.pollFirst();
```

##### 归还jedis连接
在使用完jedis连接之后,一般需要在finally调用jedis.close
```
public void close() {
    if (dataSource != null) {
      if (client.isBroken()) {
        this.dataSource.returnBrokenResource(this);
      } else {
        this.dataSource.returnResource(this);
      }
    } else {
      client.close();
    }
  }
```
这里不是真的关闭连接,而是把jedis连接换回池子里面

##### jedis连接的生命周期
请求连接的时候没有空闲连接且当前连接数量小于最大数量时创建.空闲超出最大空闲时间时销毁.

#### 一些参数
```
1）minIdle
 资源池确保最少空闲的连接数，默认值：0

2）maxIdle
 资源池允许最大空闲的连接数，默认值：8

3）maxTotal
 资源池中最大连接数，默认值：8

4）maxWaitMillis
 当资源池连接用尽后，调用者的最大等待时间，单位是毫秒，默认值：-1，建议设置合理的值

5）testOnBorrow
 向资源池借用连接时，是否做连接有效性检测，无效连接会被移除，默认值：false ，业务量很大时建议为false，因为会多一次ping的开销

6）testOnCreate
 创建新的资源连接后，是否做连接有效性检测，无效连接会被移除，默认值：false ，业务量很大时建议为false，因为会多一次ping的开销

7）testOnReturn
 向资源池归还连接时，是否做连接有效性检测，无效连接会被移除，默认值：false，业务量很大时建议为false，因为会多一次ping的开销

8） testWhileIdle
 是否开启空闲资源监测，默认值：false

9）blockWhenExhausted
 当资源池用尽后，调用者是否要等待。默认值：true，当为true时，maxWaitMillis参数才会生效，建议使用默认值

10）lifo
 资源池里放池对象的方式，LIFO
 Last In First Out 后进先出，true（默认值），表示放在空闲队列最前面，false：放在空闲队列最后面
```


