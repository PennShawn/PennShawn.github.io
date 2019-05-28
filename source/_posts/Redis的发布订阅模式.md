---
title: Redis的发布订阅模式
categories: 
 - 技术
tags:
 - redis
 - Java
---

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。与观察者模式比较相似.<br>
订阅者可以通过subscribe和psubscribe命令向redis server订阅自己感兴趣的消息类型，redis将消息类型称为通道(channel)。当发布者通过publish命令向redis server发送特定类型的消息时。订阅该消息类型的全部client都会收到此消息。这里消息的传递是多对多的。一个client可以订阅多个 channel,也可以向多个channel发送消息。


#### 订阅/退订/发布等命令
`SUBSCRIBE channel [channel...]`订阅一个或多个频道的信息.

`PSUBSCRIBE pattern [pattern...]`订阅一个或多个符合给定模式的频道.

`UNSUBSCRIBE [channel [cahnnel...]]`退订给定的频道

`PUNSUBSCRIBE [pattern [pattern]]`退订所有给定模式的频道

`PUBLISH channel message`将消息发布到追定频道

`PUBSUB subcommand[argument [argument...]]`查看订阅与发布系统状态

#### 示例
我们先订阅redisChat频道
```
redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```
然后,重新开启一个Redis客户端,在同一个频道发布消息
```
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"

(integer) 1

redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"

(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```



#### Java实例

获取Redis连接的方法
```
package redis.pub_sub;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 * @Description: Redis管理类
 * @Author: shenpeng
 * @Date: 2019-05-27
 */
public class RedisManager {

    private static JedisPool jedisPool;

    public static void init(int maxIdle, int maxTotal, int maxWaitMills, String host, int port,
            String password, int database) {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setTestOnBorrow(true);
        jedisPoolConfig.setTestOnReturn(true);
        jedisPoolConfig.setMaxIdle(maxIdle);
        jedisPoolConfig.setMaxTotal(maxTotal);
        jedisPoolConfig.setMaxWaitMillis(maxWaitMills);
        if ("".equals(password)) {
            jedisPool = new JedisPool(jedisPoolConfig, host, port, maxWaitMills, null, database);
        } else {
            jedisPool = new JedisPool(jedisPoolConfig, host, port, maxWaitMills, password,
                    database);
        }
    }

    public static Jedis getClient() {
        if (jedisPool == null) {
            System.out.println("UnInitialized");
            return null;
        }
        return jedisPool.getResource();
    }

}
```
处理Redis消息的实现类,需要继承JedisPubSub
```
package redis.pub_sub;

import redis.clients.jedis.JedisPubSub;

/**
 * @Description: 处理Redis消息的实现类
 * @Author: shenpeng
 * @Date: 2019-05-27
 */
public class MyRedisListener extends JedisPubSub {

    // 取得订阅的消息后的处理
    @Override
    public void onMessage(String channel, String message) {
        // TODO Auto-generated method stub
        System.out.println(channel + "=" + message);
    }

    // 取得按表达式的方式订阅的消息后的处理
    @Override
    public void onPMessage(String pattern, String channel, String message) {
        // TODO Auto-generated method stub
        System.out.println(pattern + ":" + channel + "=" + message);
    }

    // 初始化订阅时候的处理
    @Override
    public void onSubscribe(String channel, int subscribedChannels) {
        // TODO Auto-generated method stub
        System.out.println("初始化 【频道订阅】 时候的处理  ");
    }

    // 取消订阅时候的处理
    @Override
    public void onUnsubscribe(String channel, int subscribedChannels) {
        // TODO Auto-generated method stub
        System.out.println("// 取消 【频道订阅】 时候的处理  ");
    }

    // 初始化按表达式的方式订阅时候的处理
    @Override
    public void onPSubscribe(String pattern, int subscribedChannels) {
        // TODO Auto-generated method stub
        System.out.println("初始化 【模式订阅】 时候的处理  ");
    }

    // 取消按表达式的方式订阅时候的处理
    @Override
    public void onPUnsubscribe(String pattern, int subscribedChannels) {
        // TODO Auto-generated method stub
        System.out.println("取消 【模式订阅】 时候的处理  ");
    }

    @Override
    public void unsubscribe() {
        super.unsubscribe();
    }

    @Override
    public void unsubscribe(String... channels) {
        super.unsubscribe(channels);
    }

    @Override
    public void subscribe(String... channels) {
        super.subscribe(channels);
    }

    @Override
    public void psubscribe(String... patterns) {
        super.psubscribe(patterns);
    }

    @Override
    public void punsubscribe() {
        super.punsubscribe();
    }

    @Override
    public void punsubscribe(String... patterns) {
        super.punsubscribe(patterns);
    }
}
```
订阅
```
package redis.pub_sub;

import redis.clients.jedis.Jedis;

/**
 * @Description: 订阅Redis频道
 * @Author: shenpeng
 * @Date: 2019-05-27
 */
public class Sub {

    public static void main(String[] args) {
        MyRedisListener listener = new MyRedisListener();
        RedisManager.init(1000, 1000, 1000, "127.0.0.1", 6379, "", 1);
        Jedis jedis = RedisManager.getClient();

        jedis.psubscribe(listener, "hello*");
        System.out.println(111);
    }

}
```
发布
```
package redis.pub_sub;

import redis.clients.jedis.Jedis;

/**
 * @Description: Redis发布消息
 * @Author: shenpeng
 * @Date: 2019-05-27
 */
public class Pub {

    public static void main(String[] args) {
        RedisManager.init(1000, 1000, 1000, "127.0.0.1", 6379, "", 1);

        Jedis jedis = RedisManager.getClient();
        jedis.publish("hello_redis", "hello redis");
    }
}

```

参考来源: https://lanjingling.github.io/2015/11/20/redis-pub-sub/
          http://www.runoob.com/redis/redis-pub-sub.html
