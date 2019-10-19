---
title: 分布式锁
date: 2019-02-13 09:52:28
tags:
---

# 基本概念

目前几乎很多大型网站及应用都是分布式部署的，分布式场景中的数据一致性问题一直是一个比较重要的话题。分布式的CAP理论告诉我们“任何一个分布式系统都无法同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance），最多只能同时满足两项。”所以，很多系统在设计之初就要对这三者做出取舍。在互联网领域的绝大多数的场景中，都需要牺牲强一致性来换取系统的高可用性，系统往往只需要保证“最终一致性”，只要这个最终时间是在用户可以接受的范围内即可。

在很多场景中，我们为了保证数据的最终一致性，需要很多的技术方案来支持，比如`分布式事务`、`分布式锁`等。有的时候，我们需要保证一个方法在同一时间内只能被同一个线程执行。在单机环境中，Java中其实提供了很多并发处理相关的API，但是这些API在分布式场景中就无能为力了。也就是说单纯的Java Api并不能提供分布式锁的能力。

# 分布式锁应该具备的条件

1. 在分布式系统环境下，一个方法在同一时间只能被一个机器的一个线程执行;
2. 高可用的获取锁与释放锁;
3. 高性能的获取锁与释放锁;
4. 具备可重入特性;
5. 具备锁失效机制，防止死锁;
6. 具备非阻塞锁特性，即没有获取到锁将直接返回获取锁失败。

# 基于Redis的实现方式

## Redis命令介绍

1. ```SETNX key val``` 当且仅当key不存在时，set一个key为val的字符串，返回1；若key存在，则什么都不做，返回0
2. ```expire key timeout``` 为key设置一个超时时间，单位为second，超过这个时间锁会自动释放，避免死锁
3. ```delete key``` 删除key

## 实现思想
1. 获取锁的时候，使用setnx加锁，并使用expire命令为锁添加一个超时时间，超过该时间则自动释放锁，锁的value值为一个随机生成的UUID（用来标识一次网络请求）
2. 获取锁的时候还设置一个超时时间，若超过这个时间则放弃锁
3. 释放锁的时候，通过UUID判断是不是该锁，若是，则执行delete进行释放

## Redis操作工具类

```
import redis.clients.jedis.Jedis;

import java.util.Collections;

public class RedisTool {

    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";

    /**
     * 尝试获取分布式锁
     *
     * @param jedis      Redis客户端
     * @param lockKey    锁
     * @param requestId  请求标识
     * @param expireTime 超期时间
     * @return 是否获取成功
     */
    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }

    private static final Long RELEASE_SUCCESS = 1L;

    /**
     * 释放分布式锁
     *
     * @param jedis     Redis客户端
     * @param lockKey   锁
     * @param requestId 请求标识
     * @return 是否释放成功
     */
    public boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;

    }
}
```

## 需要加锁的业务逻辑实现

```
import cc.wangweiye.distributelock.DistributedLock;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.UUID;

public class Service2 {

    private static JedisPool pool = null;
    private RedisTool lock = new RedisTool();

    int n = 500;

    static {
        JedisPoolConfig config = new JedisPoolConfig();
        // 设置最大连接数
        config.setMaxTotal(500);
        // 设置最大空闲数
        config.setMaxIdle(8);
        // 设置最大等待时间
        config.setMaxWaitMillis(1000 * 100);
        // 在borrow一个jedis实例时，是否需要验证，若为true，则所有jedis实例均是可用的
        config.setTestOnBorrow(true);
        pool = new JedisPool(config, "127.0.0.1", 6379, 3000);
    }

    public void seckill() {
        Jedis jedis = pool.getResource();
        // 返回锁的value值，供释放锁时候进行判断
        String uuid = UUID.randomUUID().toString();

        while (true) {
            boolean locked = lock.tryGetDistributedLock(jedis, "resource", uuid, 500);

            if (locked) {
                System.out.println(Thread.currentThread().getName() + "获得了锁");
                System.out.println(--n);
                break;
            }
        }

        lock.releaseDistributedLock(jedis, "resource", uuid);
    }
}
```

## 线程执行逻辑

```
public class ThreadB extends Thread {
    private Service2 service2;

    public ThreadB(Service2 service2) {
        this.service2 = service2;
    }

    @Override
    public void run() {
        service2.seckill();
    }
}
```

## 测试代码

```
public class Test2 {
    public static void main(String[] args) {
        Service2 service2 = new Service2();
        for (int i = 0; i < 499; i++) {
            ThreadB threadB = new ThreadB(service2);
            threadB.start();
        }
    }
}
```