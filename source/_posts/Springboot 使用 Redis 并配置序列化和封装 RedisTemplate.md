---
title: Springboot 使用 Redis 并配置序列化和封装 RedisTemplate
date: 2022-09-20 11:56:49
categories: Java
tags:
    - SpringBoot
    - Redis
    - Lettuce
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Redis.jpg
---
# 前言
>为什么要配置序列化：如果不配置序列化的话，我们在redis数据库中存储的数据可能以乱码形式显示出来，不方便我们判断数据存储的正确性，说白了就是序列化以后存进去的是什么，查询出来的就是什么，否则我们的键值都会变成一串看不懂的乱码。
>![在这里插入图片描述](https://img-blog.csdnimg.cn/f019f9f1393a410593faff8681882f5a.png)




>为什么要封装RedisTemplate，因为如果不进行封装的话，大家请看，是不是有黄色的警告信息，看着起来很不舒服，RedisTemplate后面的尖括号可以填泛型，填写以后警告就消失了，但我们的类型很多，每次只能Autowired一个RedisTemplate，所以不能写尖括号内的类型，同时封装也能按照自己的习惯自定义方法，更好用。
>![在这里插入图片描述](https://img-blog.csdnimg.cn/5bbee598805d444a9743703a928edb84.png)




---


# 一、引入依赖
`只需要引入这一个就可以了，现在的版本里面包含了 lettuce ，所以不需要再额外引入 common-pool2`

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

# 二、配置yml

`因为我使用的是Springboot 3.0，所以配置层级是 spring-data-redis，如果你们的配置文件爆红，说明你们是旧版，那么把data去掉就可以了，直接是 spring-redis`

```yaml
server:
  port: 8081
spring:
  data:
    redis:
      #数据库索引
      database: 0
      #redis 服务器地址
      host: 127.0.0.1
      #redis 端口
      port: 6379
      #redis 密码 默认为空
      password:
      # 链接超时时间
      connect-timeout: 10s
      #lettuce连接池配置
      lettuce:
        pool:
          # 链接池中最小的空闲链接 默认为0
          min-idle: 0
          # 链接池中最大的空闲连接 默认为 8
          max-idle: 8
          #连接池中最大数据库链接数 默认为8
          max-active: 8
          #连接池最大阻塞等待时间 负值表示没有限制
          max-wait: -1ms

```

---


# 三、RedisConfig 配置序列化

```java
package pers.xuyijie.communityinteractionsystem.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.*;

/**
 * @author 徐一杰
 * @date 2022/3/13
 * @description redis配置
 */
@SpringBootConfiguration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //创建一个json的序列化对象
        GenericJackson2JsonRedisSerializer jackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //设置value的序列化方式json
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        //设置key序列化方式String
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //设置hash key序列化方式String
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        //设置hash value序列化json
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        // 设置支持事务
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    @Bean
    public RedisSerializer<Object> redisSerializer() {
        //创建JSON序列化器
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        //必须设置，否则无法将JSON转化为对象，会转化成Map类型
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL);
        return new GenericJackson2JsonRedisSerializer(objectMapper);
    }

}

```


---

# 四、封装RedisTemplate

`注意，我这里用的是 Java17 新增的 record 纪录类，如果你们是低版本的 JDK ，把 record 更换成 class ，然后注入RedisTemplate`
![在这里插入图片描述](https://img-blog.csdnimg.cn/292e8f4b21d0414886f353c638bff477.png)



```java
package pers.xuyijie.communityinteractionsystem.utils;

import org.springframework.data.redis.core.*;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * @author: 徐一杰
 * @date: 2022/4/3
 * @description: 对RedisTemplate进行封装
 */
@Component
@SuppressWarnings({"unchecked", "rawtypes", "ConstantConditions"})
public record RedisUtil<K, V>(RedisTemplate<K, V> redisTemplate) {

    public RedisUtil {

    }

    /**
     * 默认存活时间 2 天
     */
    private static final long DEFAULT_EXPIRE_TIME = 60 * 60 * 48;

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpireTime(K key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 指定缓存失效时间
     *
     * @param key        键
     * @param expireTime 时间(秒)
     */
    public void setExpireTime(K key, long expireTime) {
        try {
            if (expireTime > 0) {
                redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 移除指定 key 的过期时间
     *
     * @param key 键
     */
    public void removeExpireTime(K key) {
        redisTemplate.boundValueOps(key).persist();
    }

    /**
     * 获取缓存中所有的键
     *
     * @param key 键
     * @return 缓存中所有的键
     */
    public Set<K> keys(K key) {
        return redisTemplate.keys(key);
    }

    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(K key) {
        try {
            return Boolean.TRUE.equals(redisTemplate.hasKey(key));
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key删除缓
     *
     * @param keys 键
     */
    public void delete(Collection<K> keys) {
        redisTemplate.delete(keys);
    }

    /**
     * 设置分布式锁
     *
     * @param key    键，可以用用户主键
     * @param value  值，可以传requestId，可以保证锁不会被其他请求释放，增加可靠性
     * @param expire 锁的时间(秒)
     * @return 设置成功为 true
     */
    public Boolean setNx(K key, V value, long expire) {
        return redisTemplate.opsForValue().setIfAbsent(key, value, expire, TimeUnit.SECONDS);
    }

    /**
     * 设置分布式锁，有等待时间
     *
     * @param key     键，可以用用户主键
     * @param value   值，可以传requestId，可以保证锁不会被其他请求释放，增加可靠性
     * @param expire  锁的时间(秒)
     * @param timeout 在timeout时间内仍未获取到锁，则获取失败
     * @return 设置成功为 true
     */
    public Boolean setNx(K key, V value, long expire, long timeout) {
        long start = System.currentTimeMillis();
        //在一定时间内获取锁，超时则返回错误
        for (; ; ) {
            // 获取到锁，并设置过期时间返回true
            boolean acquire = redisTemplate.opsForValue().setIfAbsent(key, value, expire, TimeUnit.SECONDS);
            if (acquire) {
                return true;
            }
            //否则循环等待，在timeout时间内仍未获取到锁，则获取失败
            if (System.currentTimeMillis() - start > timeout) {
                return false;
            }
        }
    }

    /**
     * 释放分布式锁
     * @param key 锁
     * @param value 值，可以传requestId，可以保证锁不会被其他请求释放，增加可靠性
     * @return 成功返回true, 失败返回false
     */
    public boolean releaseNx(K key, V value) {
        Object currentValue = redisTemplate.opsForValue().get(key);
        boolean result = false;
        if (String.valueOf(currentValue) != null && currentValue.equals(value)) {
            result = redisTemplate.opsForValue().getOperations().delete(key);
        }
        return result;
    }

    /***********      String 类型操作           **************/
    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     */
    public void set(K key, V value) {
        try {
            redisTemplate.opsForValue().set(key, value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     */
    public void set(K key, V value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                redisTemplate.opsForValue().set(key, value);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * value增加值
     *
     * @param key 键
     * @param number 增加的值
     * @return 返回增加后的值
     */
    public Long incrBy(String key, long number) {
        return (Long) redisTemplate.execute((RedisCallback) connection -> connection.incrBy(key.getBytes(), number));
    }

    /**
     * value减少值
     *
     * @param key 键
     * @param number 减少的值
     * @return 返回减少后的值
     */
    public Long decrBy(String key, long number) {
        return (Long) redisTemplate.execute((RedisCallback) connection -> connection.decrBy(key.getBytes(), number));
    }

    /**
     * 根据key获取value
     *
     * @param key 键
     * @return 返回值
     */
    public V get(K key) {
        BoundValueOperations<K, V> boundValueOperations = redisTemplate.boundValueOps(key);
        return boundValueOperations.get();
    }

    /***********      list 类型操作           **************/

    /**
     * 将value从右边放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public void listRightPush(K key, V value) {
        ListOperations<K, V> listOperations = redisTemplate.opsForList();
        //从队列右插入
        listOperations.rightPush(key, value);
    }

    /**
     * 将value从左边放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public void listLeftPush(K key, V value) {
        ListOperations<K, V> listOperations = redisTemplate.opsForList();
        //从队列右插入
        listOperations.leftPush(key, value);
    }

    /**
     * 将list从右边放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public void listRightPushAll(K key, List<V> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 将list从左边放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public void listLeftPushAll(K key, List<V> value) {
        try {
            redisTemplate.opsForList().leftPushAll(key, value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return 返回列表中的值
     */
    public V listGetWithIndex(K key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 从list左边弹出一条数据
     *
     * @param key 键
     * @return 队列中的值
     */
    public V listLeftPop(K key) {
        ListOperations<K, V> listOperations = redisTemplate.opsForList();
        return listOperations.leftPop(key);
    }

    /**
     * 从list左边定时弹出一条
     *
     * @param key 键
     * @param timeout 弹出时间
     * @param unit 时间单位
     * @return 队列中的值
     */
    public V listLeftPop(K key, long timeout, TimeUnit unit) {
        ListOperations<K, V> listOperations = redisTemplate.opsForList();
        return listOperations.leftPop(key, timeout, unit);
    }

    /**
     * 从list右边弹出一条数据
     *
     * @param key 键
     * @return 队列中的值
     */
    public V listRightPop(K key) {
        ListOperations<K, V> listOperations = redisTemplate.opsForList();
        return listOperations.rightPop(key);
    }

    /**
     * 从list左边定时弹出
     *
     * @param key 键
     * @param timeout 弹出时间
     * @param unit 时间单位
     * @return 队列中的值
     */
    public V listRightPop(K key, long timeout, TimeUnit unit) {
        ListOperations<K, V> listOperations = redisTemplate.opsForList();
        return listOperations.leftPop(key, timeout, unit);
    }

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始下标
     * @param end   结束下标  0 到 -1 代表所有值
     * @return list内容
     */
    public List<V> listRange(K key, long start, long end) {
        try {
            ListOperations<K, V> listOperations = redisTemplate.opsForList();
            return listOperations.range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return list长度
     */
    public long listSize(K key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 下标
     * @param value 值
     */
    public void listSet(K key, long index, V value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 从lit中移除N个值为value的值
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long listRemove(K key, long count, V value) {
        try {
            return redisTemplate.opsForList().remove(key, count, value);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /***********      hash 类型操作           **************/

    /**
     * 根据key和键获取value
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public <HK, HV> HV hashGet(K key, String item) {
        HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
        return hashOperations.get(key, item);
    }

    /**
     * 获取key对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public <HK, HV> Map<HK, HV> hashMGet(K key) {
        HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
        return hashOperations.entries(key);
    }

    /**
     * 添加map到hash中
     *
     * @param key 键
     * @param map 对应多个键值
     */
    public void hashMSet(K key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 添加map到hash中，并设置过期时间
     *
     * @param key        键
     * @param map        对应多个键值
     * @param expireTime 时间(秒)
     */
    public void hashMSet(K key, Map<String, Object> map, long expireTime) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (expireTime > 0) {
                redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 向hash表中放入一个数据
     *
     * @param key   键
     * @param hKey  map 的键
     * @param value 值
     */
    public <HK, HV> void hashPut(K key, HK hKey, HV value) {
        try {
            HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
            hashOperations.put(key, hKey, value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 向hash表中放入一个数据，并设置过期时间
     *
     * @param key        键
     * @param hKey       map 的键
     * @param value      值
     * @param expireTime 时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     */
    public <HK, HV> void hashPut(K key, HK hKey, HV value, long expireTime) {
        try {
            redisTemplate.opsForHash().put(key, hKey, value);
            if (expireTime > 0) {
                redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param hKey map 的键 不能为null
     * @return true 存在 false不存在
     */
    public <HK, HV> boolean hashHasKey(K key, HK hKey) {
        HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
        return hashOperations.hasKey(key, hKey);
    }

    /**
     * 取出所有 value
     *
     * @param key 键
     * @return map 中所有值
     */
    public <HK, HV> List<HV> hashValues(K key) {
        HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
        return hashOperations.values(key);
    }

    /**
     * 取出所有 hKey
     *
     * @param key 键
     * @return map 所有的键
     */
    public <HK, HV> Set<HK> hashHKeys(K key) {
        HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
        return hashOperations.keys(key);
    }

    /**
     * 删除hash表中的键值，并返回删除个数
     *
     * @param key 键
     * @param hashKeys 要删除的值的键
     * @return 删除个数
     */
    public <HK, HV> Long hashDelete(K key, Object... hashKeys) {
        HashOperations<K, HK, HV> hashOperations = redisTemplate.opsForHash();
        return hashOperations.delete(key, hashKeys);
    }

    /***********      set 类型操作           **************/
    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     */
    public void setAdd(K key, V... values) {
        redisTemplate.opsForSet().add(key, values);
    }

    /**
     * 将set数据放入缓存，并设置过期时间
     *
     * @param key        键
     * @param expireTime 时间(秒)
     * @param values     值 可以是多个
     */
    public void setAdd(K key, long expireTime, V... values) {
        redisTemplate.opsForSet().add(key, values);
        if (expireTime > 0) {
            redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return set缓存的长度
     */
    public long setSize(K key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return Set中的所有值
     */
    public Set<V> setValues(K key) {
        SetOperations<K, V> setOperations = redisTemplate.opsForSet();
        return setOperations.members(key);
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 要查询的值
     * @return true 存在 false不存在
     */
    public boolean setHasKey(K key, V value) {
        return Boolean.TRUE.equals(redisTemplate.opsForSet().isMember(key, value));
    }

    /**
     * 根据value删除，并返回删除的个数
     *
     * @param key 键
     * @param value 要删除的值
     * @return 删除的个数
     */
    public Long setDelete(K key, Object... value) {
        SetOperations<K, V> setOperations = redisTemplate.opsForSet();
        return setOperations.remove(key, value);
    }

    /***********      zset 类型操作           **************/
    /**
     * 在 zset中插入一条数据
     *
     * @param key 键
     * @param value 要插入的值
     * @param score 设置分数
     */
    public void zSetAdd(K key, V value, long score) {
        ZSetOperations<K, V> zSetOperations = redisTemplate.opsForZSet();
        zSetOperations.add(key, value, score);
    }

    /**
     * 得到分数在 score1，score2 之间的值
     *
     * @param key 键
     * @param score1 起始分数
     * @param score2 终止分数
     * @return 范围内所有值
     */
    public Set<V> zSetValuesRange(K key, long score1, long score2) {
        ZSetOperations<K, V> zSetOperations = redisTemplate.opsForZSet();
        return zSetOperations.range(key, score1, score2);
    }

    /**
     * 根据value删除，并返回删除个数
     *
     * @param key 键
     * @param value 要删除的值，可传入多个
     * @return 删除个数
     */
    public Long zSetDeleteByValue(K key, Object... value) {
        ZSetOperations<K, V> zSetOperations = redisTemplate.opsForZSet();
        return zSetOperations.remove(key, value);
    }

    /**
     * 根据下标范围删除，并返回删除个数
     *
     * @param key 键
     * @param size1 起始下标
     * @param size2 结束下标
     * @return 删除个数
     */
    public Long zSetDeleteRange(K key, long size1, long size2) {
        ZSetOperations<K, V> zSetOperations = redisTemplate.opsForZSet();
        return zSetOperations.removeRange(key, size1, size2);
    }

    /**
     * 删除分数区间内元素，并返回删除个数
     *
     * @param key 键
     * @param score1 起始分数
     * @param score2 终止分数
     * @return 删除个数
     */
    public Long zSetDeleteByScore(K key, long score1, long score2) {
        ZSetOperations<K, V> zSetOperations = redisTemplate.opsForZSet();
        return zSetOperations.removeRangeByScore(key, score1, score2);
    }

}

```


---

# 五、controller使用RedisUtil

```java
package com.xuyijie.redisdemo.controller;

import com.xuyijie.redisdemo.utils.RedisUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @author 徐一杰
 * @date 2022/9/20 10:30
 * @description
 */
@RestController
@RequestMapping("/test")
public class TestController {

    @Autowired
    private RedisUtil<String, String> redisUtil;

    @GetMapping("/helloString/{key}")
    public String helloString(@PathVariable String key){
        redisUtil.set("key", key);
        return (String) redisUtil.get("key");
    }

    @GetMapping("/helloList")
    public List<String> helloList(){
        //向key中push一个元素“0”
        redisUtil.listLeftPush("key2", "0");
        //把一个列表push进key
        List<String> list = new ArrayList<>(Arrays.asList("1", "2"));
        redisUtil.listLeftPushAll("key2", list);
        //读取存储的列表中的全部元素，0， -1代表从第一个元素到最后一个元素
        return redisUtil.listRange("key2", 0, -1);
    }

}

```

---

# 六、操作演示

`我们先来演示一下第二个方法，直接浏览器请求，可以看到，存入并查询出了我们存入的数据`
![在这里插入图片描述](https://img-blog.csdnimg.cn/65e7183b79ba4f0b90994fb56ea16dfa.png)
`redis控制台也能查询出来`
![在这里插入图片描述](https://img-blog.csdnimg.cn/649b68f5d5d0476d99f59958559b91a9.png)
> 为什么先演示第二个方法呢，因为要用第一个向你们展示一个“小错误”，我们输入中文来测试一下
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/5347d5365be04a2992f55b333b261615.png)
> 很棒，成功存入并正确读取出来了对不对
>  <b>
>  那我们去控制台看一看，注意看，我们的键还是“key”，没有乱码，value的前半部分中文乱码，后面的String是正常显示的，说明这不是序列化的问题
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/87fca7083f4f410f958abc2643b41c51.png)
>  解决方案是，使用 redis-cli --raw 进入控制台，flushall 先清空数据，再次请求接口存储数据，查询出来的中文就出来了，但是我们还是看不懂，这和代码和redis没有关系
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/761875bad2e14440b1dc3f7a290bbfc7.png)
>   原因是电脑的 cmd 字符默认是 GBK，你们bing一下，看看怎么把cmd改为 UTF-8，好像要从注册表里面改，改好就可以正确显示了

`其实代码可以正确读取出来就说明一切正常的了，我们控制台看到乱码不必理会，这里只是提一下`


---

# 总结

