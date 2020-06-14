---
title: redis分布式锁的用法
categories: 
 - 技术
tags:
 - Java
 - redis
---

我们在项目里,会有一些不同的服务器操作同一块数据的场景,我们如果要保证并发的话,因为不实在同一个JVM中执行的逻辑,没有办法使用线程锁,这时候就可以通过redis的分布式锁来实现.

#### redis 的`SET`命令
`SET key value [EX seconds][PX millseconds] [NX|XX]`


从`Redis 2.6.12`版本开始,`SET`命令的行为可以通过一系列参数来修改：
- `EX second`: 设置key的过期时间为 second秒.<br>`set key value EX second` 等价于`setex key second value`.
- `PX millsecond`: 设置key的过期时间为millsecond毫秒.<br>`set key value PX millsecond` 等价于`psetex key millsecond value`.
- `NX`: 只有键不存在时才进行操作.<br> `set key value NX`等价于 `setnx key value`.
- `XX`: 只有键存在时才进行操作.


#### 加锁
首先是正确示例:
```
  private static final String NX = "NX";

    private static final String PX = "PX";

    private static final String LOCK_OK = "OK";

  /**
     * 加分布式锁
     * 
     * @param [randomKey, value, expireMills]
     * @return boolean
     * @Author: shenpeng
     * @Date: 2020-03-25
     */
    public boolean lock(Jedis jedis, String key, String requestId, long expireMills) {
        String result = jedis.set(key, requestId, NX, PX, expireMills);
        if (LOCK_OK.equals(result)) {
            return true;
        } else {
            return false;
        }
    }
```
##### 加锁错误示范1:
```
 /**
     * 加锁错误示范1
     * 不是原子操作
     * 
     * @param [jedis, randomKey, value, expireSeconds]
     * @return boolean
     * @Author: shenpeng
     * @Date: 2020-03-26
     */
    public boolean wrongLock1(Jedis jedis, String randomKey, String value, int expireSeconds) {
        Long result = jedis.setnx(randomKey, value);
        if (result == 1) {
            //如果这时候程序因为异常情况没有执行下去，那么这个key就不会过期了,之后就可能出现死锁
            jedis.expire(randomKey, expireSeconds);
            return true;
        } else {
            return false;
        }
    }
```
如果程序在设置过期时间时序因为异常情况没有执行下去，那么这个key就不会过期了,之后就可能出现死锁.

##### 加锁错误示范2:
```
/**
     * 加锁错误示范2
     * 这里不设置过期时间，而是用过期时间戳作value，每次加锁的时候都判断有没有过期
     * 
     * @param [jedis, randomKey, expireMills]
     * @return boolean
     * @Author: shenpeng
     * @Date: 2020-03-26
     */
    public boolean wrongLock2(Jedis jedis, String randomKey, long expireMills) {
        String newValue = String.valueOf(System.currentTimeMillis() + expireMills);
        // 如果当前锁不存在，返回加锁成功
        if (jedis.setnx(randomKey, newValue) == 1) {
            return true;
        }
        //判断上一个锁是否已经过期
        String oldTime = jedis.get(randomKey);
        if (oldTime != null && Long.valueOf(oldTime) < System.currentTimeMillis()) {
            //锁已经过期了，更新过期时间,但是要判断此时其它程序是否更新了锁
            String oldValue = jedis.getSet(randomKey, newValue);
            if (oldTime.equals(oldValue)) {
                return true;
            }
        }
        //锁没有过期，或者其他线程更新了锁，返回失败
        return false;
    }
```
这里不设置过期时间，而是用过期时间戳作value，每次加锁的时候都判断有没有过期,解决了上个示例中过期时间没有设置成功的问题.但是,有几个问题:<br>1.强制要求不同客户端的时间必须是同步的.  <br>2.多个程序执行` String oldValue = jedis.getSet(randomKey, newValue);`时,虽然只有一个程序加锁成功,但value会被set多次,不一定是加锁的程序设置的过期时间. <br>3.解锁时,任何程序都可以对key进行解锁,key和value都没有加锁客户端的标识.(在正确示例中,标识就是requestId)

#### 解锁
先看正确示例:
```
    /**
     * 解锁
     * 
     * @param [jedis, key, requestId]
     * @return boolean
     * @Author: shenpeng
     * @Date: 2020-03-26
     */
    public boolean unlock(Jedis jedis, String key, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(key),
                Collections.singletonList(requestId));

        if (UN_LOCK_OK.equals(result)) {
            return true;
        }
        return false;
    }
```
我们将Lua代码传到jedis.eval()方法里，并使参数KEYS[1]赋值为lockKey，ARGV[1]赋值为requestId。eval()方法是将Lua代码交给Redis服务端执行。<br>
那么这段Lua代码的功能是什么呢？其实很简单，首先获取锁对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）。那么为什么要使用Lua语言来实现呢？因为要确保上述操作是原子性的。关于非原子性会带来什么问题，可以阅读【解锁代码-错误示例2】 。那么为什么执行eval()方法可以确保原子性，源于Redis的特性，下面是官网对eval命令的部分解释：
`简单来说，就是在eval命令执行Lua代码的时候，Lua代码将被当成一个命令去执行，并且直到eval命令执行完成，Redis才会执行其他命令。`

##### 解锁的错误示例1
最常见的解锁代码就是直接使用jedis.del()方法删除锁，这种不先判断锁的拥有者而直接解锁的方式，会导致任何客户端都可以随时进行解锁，即使这把锁不是它的。
```
 public void wrongUnlock1(Jedis jedis, String key) {
        jedis.del(key);
    }
```

##### 解锁的错误示例2
```
/**
     * 解锁的错误示例12
     * 不是原子操作
     * 
     * @param [jedis, key, requestId]
     * @return void
     * @Author: shenpeng
     * @Date: 2020-03-26
     */
    public void wrongUnlock2(Jedis jedis, String key, String requestId) {
        if (requestId.equals(jedis.get(key))) {
            //如果这时候，另一个程序加锁了，这个锁立马又被删了
            jedis.del(key);
        }
    }
```
这种操作不是原子的,如果程序A判断锁可以删除后,锁过期了,程序B加上了锁,这时候程序A就有可能会把B刚加的锁删了.


参考来源:https://blog.csdn.net/yb223731/article/details/90349502
