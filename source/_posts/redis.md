---
title: redis概述
date: 2019-01-20
---

[TOC]


### redis概述
@(ppt) 

#### redis是什么
Redis 是完全开源免费的，是一个高性能的key-value数据库

#### redis的优缺点
- 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性

### 安装和启动Redis


#### 安装Redis
```
$ wget http://download.redis.io/releases/redis-5.0.0.tar.gz
$ tar xzf redis-5.0.0.tar.gz
$ cd redis-5.0.0
$ make
$ make test
```

#### 启动Redis服务端
```
/* 1.默认启动 默认端口号是6379*/
$ src/redis-server 
/* 2.运行启动 src/redis-server --key1 value1 --key2 value2 */
$ src/redis-server --port 6380 
/* 3.配置文件启动(推荐) src/redis-server conf路径*/
$ src/redis-server redis.conf
```

#### 启动Redis客户端
```
/* redis-cli -h {host} -p {port} */
$ src/redis-cli -h 127.0.0.1 -p 6380
```

#### 停止Redis服务
```
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> shutdown
not connected>
```

### 数据结构
[命令参考](http://doc.redisfans.com/)

#### String（字符串）
<b><b>常用命令</b></b>
| 操作 | 命令 | 
| :--: | :----:|
| 设置值 | `set key value [ex seconds] [px milliseconds] [nx\|xx]` |
| 获取值 | `get key`  |
| 批量设置 | `mset key val [key2 val2]` |
| 批量获取 | `mget key` |
| 字符串长度 | `strlen key` |
| 追加 | `append key value` |

- nx 键必须不存在才能设置成功，用户添加，同命令 `setnx key value` 简单的锁
- xx 键必须存在才能设置成功，用于更新

**示例**
```
127.0.0.1:6379> set name zhangxinxin
OK
127.0.0.1:6379> get name
"zhangxinxin"
127.0.0.1:6379> mset age 27 sex female
OK
127.0.0.1:6379> mget age sex
1) "27"
2) "female"
127.0.0.1:6379> strlen name
(integer) 11
127.0.0.1:6379> append name ' hello'
(integer) 17
127.0.0.1:6379> get name
"zhangxinxin hello"
127.0.0.1:6379> setnx name lucy
(integer) 0
127.0.0.1:6379> get name
"zhangxinxin hello"
127.0.0.1:6379>
```
<b>使用场景</b>
1. 缓存功能
```sequence
Web服务->缓存层(Redis): hit
缓存层(Redis)-->Web服务: return
缓存层(Redis)->存储层(Mysql): miss
存储层(Mysql)-->缓存层(Redis): write cache
存储层(Mysql)-->Web服务: return
```
2. 计数 文章浏览量，视频播放量等
3. 限速 验证码一分钟不超过几次，进博会push一天推送一次等

#### Hash（哈希）
> 哈希类型中的映射关系叫做field-value

<b>常用命令</b>
| 操作 | 命令 | 
| :--: | :----:|
 |将哈希表key中的域field的值设为 value | `hset key field value` |
| 返回哈希表key中域field 的值 |  `hget key field` |
| 删除field | `hdel key field` |
| 计算field个数 | `hlen key` |
| 批量设置 | `hmset key field value [field value ...]`   |
| 批量获取 | `hmget key field [field ...]` |
| 判断域field是否存在 | `hexists key field` |
| 获取所有field | `hkeys key` |
| 获取所有value | ` hvals key` |
| 获取所有field-value | `hgetall key` |
| 自增域的值 | `hincrby key field increment` |

<b>示例<b/>
```
127.0.0.1:6379> hset driver01 name coco
(integer) 1
127.0.0.1:6379> hmset driver01 mobile 15158117762 number 362324198105255366 city 北京
OK
127.0.0.1:6379> hexists driver01 name
(integer) 1
127.0.0.1:6379> hgetall driver01
1) "name"
2) "coco"
3) "mobile"
4) "15158117762"
5) "number"
6) "362324198105255366"
7) "city"
8) "\xe5\x8c\x97\xe4\xba\xac"
127.0.0.1:6379> exit
$ src/redis-cli --raw  //显示中文的话 加--raw参数
127.0.0.1:6379> hget driver01 city
北京
```
<b>使用场景</b>
1. 关系型数据库中的某表某字段
| id | name | age | city |
| :--: | :----:| :--: | :----:|
| 1001 | zhangsan | 20 | beijing|
| 1002 | lisi | 30 | shanghai |
| 1003 | wangwu | 40 | chengdu |

转化为redis存储
```
127.0.0.1:6379> sadd userlist 1001 1002 1003
127.0.0.1:6379> hmset user#1001 name zhangsan age 20 city beijing
OK
127.0.0.1:6379> hmset user#1002 name lisi age 30 city shanghai
OK
127.0.0.1:6379> hmset user#1003 name wangwu age 40 city chengdu
OK
127.0.0.1:6379> hget user#1002 name
"lisi"
127.0.0.1:6379> hmget user#1003 name age
1) "wangwu"
2) "40"
```


#### List（列表）
> 列表用来存储多个有序的字符串，主要操作是插入和弹出，以及获取元素

<b>常用命令</b>
| 操作 | 命令 | 
| :--: | :----:|
| 插入 |  `rpush key value [value ...]` or `lpush key value [value ...]` |
| 弹出 |  `rpop key` or `lpop key` |
| 查找 | `lrange key start end` |
| 获取指定索引下标元素 | `lindex key index` |
| 获取列表长度 | `llen key` |
| 修改 | `lset key index newValue` |
| 阻塞 | `blpop key [key ...] timeout` or `brpop key [key ...] timeout` |
| 指定删除 | `lrem key count value` |
- count > 0 从左到右删除最多count个value元素
- count < 0 从右到左删除最多count个value元素
- count = 0 删除所有value元素
```
127.0.0.1:6379> lpush uid 101 102 103 104
(integer) 4
127.0.0.1:6379> lrange uid 0 -1
1) "104"
2) "103"
3) "102"
4) "101"
127.0.0.1:6379> rpush uid 110 111
(integer) 6
127.0.0.1:6379> lrange uid 0 -1
1) "104"
2) "103"
3) "102"
4) "101"
5) "110"
6) "111"
127.0.0.1:6379> llen uid
(integer) 6
127.0.0.1:6379> lindex uid 2
"102"
127.0.0.1:6379> lpop uid
"104"
127.0.0.1:6379> rpop uid
"111"
```
<b>使用场景</b>
1. lpush + lpop = Stack(栈)
2. lpush + rpop = Queue(队列)
3. lpush + brpop = Message Queue (消息队列)

#### Set（集合）
> 集合中不允许有重复元素，而且集合中的元素是无序的

<b>常用命令</b>
添加元素	`sadd key element [element ...]`
删除元素 `srem key element [element ...]`
判断元素是否在集合中 `sismember key element`
计算个数 `scard key`
获取所有元素 `smembers key`
交集 `sinter key [key ...]`
差集 `sdiff key [key ...]`
并集 `sunion key [key ...]`
交集 并集 差集保存到新key
`sinterstore destination key [key ...]` 
`sdiffstore destination key [key ...]` 
`sunionstore destination key [key ...]` 

```

```
<b>使用场景</b>
给用户加标签

#### SortedSet（有序集合）
> 有序集合中的元素可以排序。它给每个元素设置一个分数（score）作为排序的依据

<b>常用命令</b>
添加成员和分数  `zadd key score member [score member ...]`
计算成员个数  `zcard key`
获取成员分数 `zscore key member`
计算成员的排名 `zrank key member`  `zrevrank key member`
返回指定排名范围的成员 	
`zrange key start end [withscores]`
`zrevrange key start end [withscores]`		
删除某个成员 `zrem key member`
增减 `zincrby key increment member`
<b>使用场景</b>
用户赞数排序 
文章按照发布时间排序
等等排序

### 键的全局命令
Redis常用的数据结构有5种，数据结构是键值对中的值。Redis中的键有一些全局命令
```
127.0.0.1:6379> keys *	/* 查看所有键 支持*前*后加关键字符 */
127.0.0.1:6379> dbsize	/* 键总数 */
127.0.0.1:6379> exists key	/* 检查键是否存在 */
127.0.0.1:6379> del key [key2 ...] 	/* 删除键 */
127.0.0.1:6379> expire key second /* 设置键过期时间 */
127.0.0.1:6379> ttl key  /* >=0:剩余过期时间 -1:没设 -2:键不存在 */
127.0.0.1:6379> type key  /* 查看数据结构类型 */
/* 值不是整数,返回错误；值是整数，返回自增结果；键不存在，按照值为0自增，返回1*/
127.0.0.1:6379> incr key  /* decr 自减 */
127.0.0.1:6379> incrby key increment  /* 自增指定数字 decrby自减指定数字 */  
```


### 使用jedis
安装java的redis驱动包jedis即可
1. 添加pom依赖
```
<!-- redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
```
2. 代码调用
```
public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("127.0.0.1", 6279);
        String key = "str";
        jedis.set(key, "hello world");
        System.out.println("redis 存储的字符串为: "+ jedis.get(key));
    }
```
 
----------


<b>其他</b>
掌握以上基本功能基本满足我们对redis的日常使用，除此之外，redis还有很多其他功能和特性，例如`redis持久化`     ` redis的复制`  `redis内存管理`  `哨兵机制`   `集群` `pipeline`  `事务`等等 有兴趣的小伙伴可以深入学习，附上推荐书籍和网站。
<b>推荐书籍&推荐网站</b>
- 【Redis开发与运维】 付磊 张益军 著
- 【Redis设计与实现】 黄健宏 著
- [Redis官网](https://redis.io/)
- [Redis命令参考](http://redisdoc.com/)


