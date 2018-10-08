---
layout: post
title: 'Redis系列(二):Redis的数据类型及命令操作'
comments: true
date: 2018-10-5 15:30:11
categories: [DataBase]
tags: [Redis,学习笔记]
---

<!--more -->

<center>
<img src="learn-redis-2\redis-white.png" width="250px" />
</center>

## Redis 中常用命令

Redis 官方的[文档](https://redis.io/documentation)是英文版的，当然网上也有大量的中文翻译版,例如:[Redis 命令参考](http://doc.redisfans.com/)。这里只列举常用到几个基本命令。

| 命令               | 行为                                                     |
| ------------------ | -------------------------------------------------------- |
| set key value      | 设置 key 值为 value                                      |
| get key            | 读取 key 的值                                            |
| del key            | 删除 key                                                 |
| expire key seconds | 设置 key 的生存时间（seconds 秒后自动删除）              |
| ttl key            | 查看 key 剩余生存时间                                    |
| exists key         | 判断 key 是否存在                                        |
| ping               | 测试与服务端是否联通                                     |
| keys *             | 匹配数据库中所有 key                                     |
| dbsize             | 查询当前数据库中 key 的数量                              |
| info               | 返回关于 Redis 服务器的各种信息和统计数值                |
| flushdb            | 清空当前数据库中的所有 key                               |
| flushall           | 清空整个 Redis 服务器的数据( 删除所有数据库的所有 key ） |
| quit               | 请求服务器关闭与当前客户端的连接( 断开连接 )             |

## Redis数据类型简介

| 数据类型   | 存储的值               | 读写能力                                                   |
| ---------- | ---------------------- | ---------------------------------------------------------- |
| String     | 字符串，整数或浮点数   | 对字符串或一部分字符串执行操作；对整数进行自增和自减操作等 |
| Hash       | 包含键值对的无序散列表 | 对单个 元素进行增、删、改；获取所以的键值对等              |
| List       | 链表上的节点字符串元素 | 推入、弹出元素；修剪、查找、移除元素等                     |
| Set        | 各不相同的字符串元素   | 对单个 元素进行增、删、改；计算集合 交，并补集等           |
| Sorted Set | 带分数的有序集合       | 对单个 元素进行增、删、改；按照分数范围查元素等            |



## 字符串类型（String）

####  redis string 介绍

虽然 redis 是用 C 编写的，但是在 redis 中没有使用 C 语言的字符串，而是自定义了一个数据结构叫 SDS (simple dynamic string) ——简单动态字符串。是可以修改的，类似java中的 ArrayList。同时它也不像 C 中的字符串那样遇到 `\0`字符就认为字符串结束，他不会对存储进去的字符串数据进行任何处理，因此 redis 中的字符串也是二进制安全的。

redis 的 string 类型可以包含任意数据，包括图片等二进制或者序列化的对象等。单个 `value` 的值最大上限为 `1G `字节。

#### 纯字符串操作命令

**注**：*`#`后面是命令的注释，不可执行*

```bash
127.0.0.1:6379> set key1 'hello redis' # 存值
OK
127.0.0.1:6379> get key1 # 取值
"hello redis"
127.0.0.1:6379> getset key1 redis # 将给定key1的值设为redis，并返回key1的旧值(old value)
"hello redis"
127.0.0.1:6379> get key1
"redis"
127.0.0.1:6379> del key1 # 删除 key1
(integer) 1
```

#### 整数自增自减操作命令

```bash
127.0.0.1:6379> set key2 10 # 存入一个值为10的整数字符串
OK
127.0.0.1:6379> incr key2 # 自增
(integer) 11
127.0.0.1:6379> incr key2 # 自增
(integer) 12
127.0.0.1:6379> incrby key2 5 # 自增指定数值 -- 5
(integer) 17
127.0.0.1:6379> decr key2 # 自减
(integer) 16
127.0.0.1:6379> decr key2 # 自减
(integer) 15
127.0.0.1:6379>  decrby key2 5 # 自减指定数值 -- 5
(integer) 10
```

#### 其他命令

```bash
127.0.0.1:6379> set key1 redis
OK
127.0.0.1:6379> append key1 hello # 将 hello 追加到 key1 原来的值的末尾,放回追加后字符串长度
(integer) 10
127.0.0.1:6379> get key1
"redishello"
127.0.0.1:6379> strlen key1 # 返回 key1 所储存的字符串值的长度
(integer) 10
127.0.0.1:6379> mset key1 v1 key2 v2 key3 v3 # 批量同时设置一个或多个 key-value 对
OK
127.0.0.1:6379> mget key1 key2 key3 # 返回所有(一个或多个)给定 key 的值
1) "v1"
2) "v2"
3) "v3"
```

#### 应用场景

- 商品编号，订单号采用string 的递增数字特性生成

## 散列类型（Hash）

####  redis hash介绍

 hash 叫散列类型。等价于Java 中的 HashMap。但是在 redis 中 hash 的 key 必须是 string 类型。不支持其他类型。这个特性非常适合存储对象。因为一个对象可以有很多属性，存储起来就是键值对形式的。在 Reids 中，每个 Hash 可以存储多达 4 亿个键值对。

![图示](learn-redis-2\hash.png)

#### 相关操作命令

```bash
127.0.0.1:6379> hset user name zhangsan # 使用 hset 为 user 添加一个键值对 name = zhangsan
(integer) 1
127.0.0.1:6379> hset user age 18 # 使用 hset 为 user 添加一个键值对 age = 18
(integer) 1
127.0.0.1:6379> hget user name # 使用 hget 获取 user 中键为 name 的值
"zhangsan"
127.0.0.1:6379> hget user age # 使用 hget 获取 user 中键为 age 的值
"18"
127.0.0.1:6379> hgetall user # 使用 hgetall 获取 user 中所有的键值对
1) "name"
2) "zhangsan"
3) "age"
4) "18"
127.0.0.1:6379> hmset user name lisi age 20 # 使用 hmset 为 user 批量添加键值对
OK
127.0.0.1:6379> hmget user name age # 使用 hmget 批量获取 user 中键的值
1) "lisi"
2) "20"
127.0.0.1:6379> hdel user name# 使用 hdel 删除 user 一个（或多个）键值对
(integer) 1
127.0.0.1:6379> hexists user name # 使用 hexists 判断 user 中 name 元素是否存在
(integer) 0
127.0.0.1:6379> hexists user age # 使用 hexists  user 中 age 元素是否存在
(integer) 1
127.0.0.1:6379> hkeys user # 使用 hkeys 只获得 user 中的字段名
1) "age"
127.0.0.1:6379> hvals user # 使用 hvals 只获得 user 中的字段值
1) "20"
127.0.0.1:6379> hlen user # 使用 hlen 获得  user 中字段（键值对）数量
(integer) 1
```

#### 其他特性

Redis 中的 Hash 结构有`扩容`和`缩容`特性，扩容主要应用在当 hash 内部比较拥挤的时候，容易产生 hash 碰撞，这时需要扩容 hash 。申请新的两倍大小的数组；而缩容与扩容恰恰相反，虽然原理一样，但是申请的新数组要比旧的小一倍。

#### 应用场景

- 保存大量的对象数据

## 列表类型（List)

#### redis list介绍
在 java 中，列表类型有两种，一种是 ArrayList，实现方式是`数组`，所以根据索引查询数据速度快，而插入或者删除某个元素涉及到位移操作，会比较慢；另一种是 LinkedList，实现方式是`双向链表（double linked list）`，每个元素都记录着前后元素的指针。所以在插入或删除某个元素时只需要更改该元素的前后指针指向就行，非常快，但是在查询上需要从头索引，特别是当数据量大的时候，索引起来还是比较慢的。 

在 Redis 中的 List 类型，其内部使用的是`双向链表`实现的,所以它具有双向链表具有的相关特性。其常用操作就是向列表两端添加或删除元素。这使得 List 既可以当做`栈（先进后出）`来使用，也可以当做`队列（先进先出）`来使用。

#### 相关操作命令

```bash
127.0.0.1:6379> lpush list 1 2 3 4 # 使用 lpush 将 1 2 3 4 依次插入到 list 的左端
(integer) 4
127.0.0.1:6379> rpush list 5 6 7 8 # 使用 rpush 将 5 6 7 8 依次插入到 list 的右端
(integer) 8
127.0.0.1:6379> lrange list 0 -1 # 使用 lrange 获取 指定区间上所有值（0 -1 表示获取全部）
1) "4"
2) "3"
3) "2"
4) "1"
5) "5"
6) "6"
7) "7"
8) "8"
127.0.0.1:6379> lpop list # 使用 lpop 弹出 list 左端的一个值，并返回弹出的值
"4"
127.0.0.1:6379> lpop list 
"3"
127.0.0.1:6379> rpop list # 使用 rpop 弹出 list 右端的一个值，并返回弹出的值
"8"
127.0.0.1:6379> rpop list
"7"
127.0.0.1:6379> lrange list 0 -1
1) "2"
2) "1"
3) "5"
4) "6"
127.0.0.1:6379> llen list # 使用 llen 获取 list 中元素个数
(integer) 4
```

#### 应用场景

- 商品，博客，文章下面的评论列表。





## 集合类型（Set）

####  redis set 介绍

redis 中的 set 类型和 java 中的 HashSet 类似，其底层都是用HashMap 实现的，只不过所有的 value 都指向同一个对象。在 set 中，没有重复的元素。并且没有顺序。其常用的操作就是向集合中加入或删除一个元素、判断某个元素是否在集合中。另外 redis 还提供了多个集合之间的 交集、并集、差集的运算。

#### 相关操作命令

```bash
127.0.0.1:6379> sadd set a b c 1 2 3  # 使用 sadd 将 a b c 1 2 3 添加到 set 集合中
(integer) 6
127.0.0.1:6379> sadd set a b 2  # 添加重复元素，返回成功添加 0 个，说明 set 中元素不重复
(integer) 0
127.0.0.1:6379> srem set a b 1  # 使用 srem 删除 set 集合中的 a b 1 三个元素
(integer) 3
127.0.0.1:6379> smembers set  # 使用 smembers 获取 set 集合中所以元素
1) "2"
2) "c"
3) "3"
127.0.0.1:6379> sismember set a  # 使用 sismember 判断 a 是否在 set 集合中
(integer) 0
127.0.0.1:6379> sismember set c  # 使用 sismember 判断 c 是否在 set 集合中
(integer) 1
```

#### 集合间的相关运算

##### 集合的并集运算 A ∪ B

![并集](learn-redis-2\buji.png)



```bash
127.0.0.1:6379> sadd seta 1 2 3
(integer) 3
127.0.0.1:6379> sadd setb 3 4 5
(integer) 3
127.0.0.1:6379> sunion seta setb # 使用 sunion 计算 seta 和 setb 的并集
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

##### 集合的交集运算 A ∩ B

![交集](learn-redis-2\jiaoji.png)

```bash
127.0.0.1:6379> sadd seta 1 2 3
(integer) 3
127.0.0.1:6379> sadd setb 3 4 5
(integer) 3
127.0.0.1:6379> sinter seta setb # 使用 sinter 计算 seta 和 setb 的交集
1) "3"
```

##### 集合的差集运算 A - B

![差集](learn-redis-2\caji.png)

```bash
127.0.0.1:6379> sadd seta 1 2 3
(integer) 3
127.0.0.1:6379> sadd setb 3 4 5
(integer) 3
127.0.0.1:6379> sdiff seta setb # 使用 sdiff 计算 seta - setb (属于seta 但不属于 setb)
1) "1"
2) "2"
127.0.0.1:6379> sdiff setb seta # 使用 sdiff 计算 setb - setb (属于setb 但不属于 seta)
1) "4"
2) "5"
```



## 有序集合类型 (sorted set)

#### redis sorted set介绍

`SortedSet(zset)`是 redis 提供的一个非常特别的数据结构，它在集合的基础上为每一个元素都关联了一个分数，这相当于 java 中的 `Map<String,Double>`，可以给每一个元素赋予一个`score`权重。另一方面又像`TreeSet`，内部的元素会按照权重`score`进行排序。 

#### 相关操作命令

```bash
127.0.0.1:6379> zadd board 81 java 75 python 90 c++ # 使用 zadd 增加一到多个value/score对，score 放在前面
(integer) 3
127.0.0.1:6379> zscore board java # 使用 zscore 获取 java 的 score
"81"
127.0.0.1:6379> zrange board 0 -1 # 使用 zrange 获取指定区间（0 -1 表示全部）上的降序排名
1) "python"
2) "java"
3) "c++"
127.0.0.1:6379> zrange board 0 -1 withscores # 带上 winthscores 可以一并获取元素的 score
1) "python" 
2) "75"
3) "java"
4) "81"
5) "c++"
6) "90"
127.0.0.1:6379> zrevrange board 0 -1 withscores # 使用 zrevrange 获取指定区间（0 -1 表示全部）上的升序排名
1) "c++"
2) "90"
3) "java"
4) "81"
5) "python"
6) "75"
127.0.0.1:6379> zrangebyscore board -inf +inf withscores # 使用 zrangebyscore 获取 负无穷（-inf）到 正无穷（+inf）区间上所以元素的降序排名
1) "python"
2) "75"
3) "java"
4) "81"
5) "c++"
6) "90"
127.0.0.1:6379> zrevrangebyscore board +inf -inf withscores # 使用 zrevrangebyscore 获取正无穷（+inf）到 负无穷（-inf）区间上所以元素的升序排名 
1) "c++"
2) "90"
3) "java"
4) "81"
5) "python"
6) "75"
127.0.0.1:6379> zcard board # 使用 zcard 计算 board 集合的元素个数
(integer) 3
127.0.0.1:6379> zrem board java python # 使用 zrem 删除 board 集合中的一个或多个元素 
(integer) 2
```

#### 应用场景

- 商品销售，软件下载等各种排行榜

## 参考

- [通俗易懂的Redis数据结构基础教程](https://juejin.im/post/5b53ee7e5188251aaa2d2e16)

- [[Redis基础：基本介绍、redis的应用场景、五种数据类型、持久化操作、主从模式]](https://segmentfault.com/a/1190000008645186)
- [Redis 命令参考](http://doc.redisfans.com/)

