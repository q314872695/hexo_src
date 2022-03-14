---
title: redis常用命令以及使用java操作redis
date: 2020-05-27 22:40:24
tags:
- java
- redis
---

# 命令操作
**字符串类型**
- 存储：`set key value`
- 获取：`get key`
- 删除：`del key`


**哈希类型**
- 存储：`hset key field value`
- 获取：
	- `hget key field`：获取指定的field对应的值
	- `hgetall key`：获取所有的field和value
- 删除：`hdel key field`

**列表类型**
- 存储：
	- `lpush key value`：将元素加入列表头部
	- `rpush key value`：将元素加入列表尾部
- 获取：`lrange key start end `：范围获取
	- 获取列表全部元素`lrange key 0 -1`
- 删除：
	- `lpop key`： 删除列表表头的元素，并将元素返回
	- `rpop key`： 删除列表表尾的元素，并将元素返回

**集合类型**
- 存储：`sadd key value`
- 获取：`smembers key` 获取set集合中所有元素
- 删除：`srem key value`
- 判断是否是集合成员：`sismember key value` 判断value是否是集合key中的元素

**有序集合类型**
- 存储：`zadd key score value`
- 获取：`zrange key start end [withscores]` 有`withscores`时同时也会显示对应的分数，不写则不显示
- 删除：`zrem key value`

**通用命令**
- `keys *`：查询所有的键
- `type key`：获取键对应的value的类型
- `del key`：删除指定的key value

# java 操作redis
1. 导入jar包
	1. `jedis-2.7.0.jar`
	2. `commons-pool2-2.3.jar` 使用连接池时需要
2. 使用(调用的方法基本和命令名称一样)
```java
public class Demo1 {
    public static void main(String[] args) {
        // //如果使用空参构造，默认值 "localhost",6379端口
        Jedis jedis = new Jedis();
        jedis.set("name", "AAA");
        System.out.println(jedis.get("name"));
        jedis.close();
    }
}
```
3. 使用jedis连接池： JedisPool
```java
public class Demo2 {
    public static void main(String[] args) {
        JedisPool jedisPool = new JedisPool();
        Jedis jedis = jedisPool.getResource();
        String name = jedis.get("name");
        System.out.println(name);
        jedis.close();
    }
}
```






