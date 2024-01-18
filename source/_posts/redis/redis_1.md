---
title: redis_1
categories:
- redis
tags:
- redis
---
<meta name="referrer" content="no-referrer"/>

## 内容包括

- docker启动redis
- docker进入redis-cli
- 认识NoSql
- redis常用命令

<!--more-->

## docker运行redis

### yaml文件准备

~~~yaml
version: '3.9'

services:
  redis:
    image: redis:latest
    container_name: hdb_redis
    restart: always
    ports:
      - "6379:6379"
    networks:
      - backend
    command: redis-server --requirepass passwd
    volumes:
          - ./redis_data:/data  # 将数据挂载到本地目录
  redis_backup:
    image: redis:latest
    command: sh -c 'exec redis-server --requirepass passwd --save 3600 1'  # 每小时备份一次
    volumes:
      - ./redis_data:/data  # 使用相同的挂载点
    depends_on:
      - redis
    networks:
      - backend
~~~

确保yaml文件同目录下有`redis_data`文件夹，备份将在`redis_data`下

~~~bash
sudo docker exec -it hdb_redis redis-cli -a passwd BGSAVE
~~~

### 启动

启动环境

~~~bash
sudo docker compose -f docker-compose.yml up -d
~~~

停止服务而不删除相关容器

```bash
sudo docker compose -f docker-compose.yml stop
```

重新启动已经停止的服务

~~~bash
sudo docker-compose -f docker-compose.yml start
~~~

关闭通过 Docker Compose 启动的服务

~~~bash
sudo docker-compose -f docker-compose.yml down
~~~

### 进入Redis终端

```
docker exec ：在运行的容器中执行命令
# 语法
-d :分离模式: 在后台运行
-i :即使没有附加也保持STDIN 打开
-t :分配一个伪终端
```

`redis-cli`表示运行一个redis客户端

~~~bash
docker exec -it 29f48e38c77b redis-cli
# 推荐
sudo docker exec -it hdb_redis redis-cli
~~~

da45019bf760是redis的容器id

退出

~~~bash
exit
~~~

~~~bash
ctr+D
~~~

### 密码认证

~~~bash
auth passwd
~~~

## 认识NoSQL

### 结构化与非结构化

- 传统关系型数据库是结构化数据，每一张表都有严格的约束信息：字段名.字段数据类型.字段约束等等信息，插入的数据必须遵守这些约束：
- 而NoSql则对数据库格式没有严格约束，往往形式松散，自由。

### 关联和非关联

- 传统数据库的表与表之间往往存在关联，例如外键
- 而非关系型数据库不存在关联关系，要维护关系要么靠代码中的业务逻辑，要么靠数据之间的耦合

### 查询方式

传统关系型数据库会基于Sql语句做查询，语法有统一标准；

而不同的非关系数据库查询语法差异极大，五花八门各种各样。

### 事务

统关系型数据库能满足事务ACID的原则。

而非关系型数据库往往不支持事务，或者不能严格保证ACID的特性，只能实现基本的一致性。

> 1. **原子性 (Atomicity):** 事务是原子的，要么全部执行，要么全部不执行。如果事务中的任何一部分操作失败，整个事务将被回滚到初始状态。
> 2. **一致性 (Consistency):** 事务使数据库从一个一致的状态转移到另一个一致的状态。这意味着事务执行后，数据库必须仍然满足所有的完整性约束。
> 3. **隔离性 (Isolation):** 多个事务可以并发执行，但其结果必须与按某种顺序串行执行它们的结果一致。隔离性确保一个事务的执行不会受到其他事务的干扰。
> 4. **持久性 (Durability):** 一旦事务提交，其结果应该是永久性的，即使在系统故障的情况下也不会丢失。

### 小结

![Nosql](https://gitee.com/hollis7/pictures/raw/master/2024/01/16/27857_Nosql.png)

## 认识Redis

Redis诞生于2009年全称是**Remote Dictionary Server**，远程词典服务器，是一个基于**内存**的键值型NoSQL数据库

- 键值（key-value）型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟，速度快（**基于内存**.IO多路复用.良好的编码-c语言）。
- 支持数据持久化（断电-定期备份磁盘）
- 支持主从集群.分片集群
- 支持多语言客户端

## 初始redis

### 设值取值

~~~bash
set name jack
get name
~~~

### 选择数据库

~~~bash
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> set age 24
~~~

## redis命令文档

Redis是典型的key-value数据库，key一般是字符串，而value包含很多不同的数据类型：

Redis为了方便我们学习，将操作不同数据类型的命令也做了分组，在[官网](https://redis.io/commands)可以查看到不同的命令。

或者使用命令

~~~bash
help @hash
~~~

## redis通用命令

通用指令是部分数据类型的，都可以使用的指令，常见的有：

- KEYS：查看符合模板的所有key
- DEL：删除一个指定的key
- EXISTS：判断key是否存在
- EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除
- TTL：查看一个KEY的剩余有效期

### keys

~~~bash
# 不建议
127.0.0.1:6379> keys a*
1) "age"
~~~
### del

~~~bash
127.0.0.1:6379> del age
(integer) 1
127.0.0.1:6379> get age
(nil)
~~~
### mset

~~~bash
# 多键值对赋值
mset k1 v1 k2 v2 k3 v3
# 结果
127.0.0.1:6379> keys *
1) "k3"
2) "k1"
3) "k2"
4) "name"
~~~
### del 2

~~~bash
# 批量删除
127.0.0.1:6379> del k1 k2 k3
(integer) 3
~~~
### exists

~~~bash
# Determines whether one or more keys exist.
127.0.0.1:6379> exists age
(integer) 0
127.0.0.1:6379> exists name
(integer) 1
~~~

### expire

设置key的过期时间（秒）

~~~bash
expire k1 20
~~~

~~~bash
# 显示-2表示已经删除
127.0.0.1:6379> TTl k1
(integer) -2
~~~

`-1`代表永久有效

~~~bash
127.0.0.1:6379> TTL age
(integer) -1
~~~

## String类型

String类型，也就是字符串类型，是Redis中最简单的存储类型。

其value是字符串，不过根据字符串的格式不同，又可以分为3类：

- string：普通字符串
- int：整数类型，可以做自增、自减操作
- float：浮点类型，可以做自增、自减操作

不管是哪种格式，底层都是**字节数组**形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过512m.

### String的常见命令

String的常见命令有：

- SET：添加或者修改已经存在的一个String类型的键值对
- GET：根据key获取String类型的value
- MSET：批量添加多个String类型的键值对
- MGET：根据多个key获取多个String类型的value
- INCR：让一个整型的key自增1
- INCRBY:让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2
- INCRBYFLOAT：让一个浮点类型的数字自增并指定步长
- SETNX：添加一个String类型的键值对，前提是这个key不存在，否则不执行
- SETEX：添加一个String类型的键值对，并且指定有效期

### INCR

~~~bash
INCR age
~~~

### INCRBY

自增3

~~~bash
127.0.0.1:6379> INCRBY age 3
(integer) 28
127.0.0.1:6379> INCRBY age 3
(integer) 31
127.0.0.1:6379> 
~~~

自减直接负数

### INCRBYFLOAT

~~~bash
127.0.0.1:6379> set score 4.5
OK
127.0.0.1:6379> INCRBYFLOAT score 3.4
"7.9"
~~~

> 这里注意score最好介于-128~128之间，超过的话，incrbyfloat会发生位数异常

### SETNX

添加一个String类型的键值对，前提是这个key不存在，否则不执行

~~~bash
127.0.0.1:6379> setnx name lush
(integer) 0
~~~

### SETEX

Sets the string value and expiration time of a key. 

~~~bash
127.0.0.1:6379> SETEX name 30 jack
OK
127.0.0.1:6379> ttl name
(integer) 22
~~~

## Key结构

**Redis没有类似MySQL中的Table的概念，我们该如何区分不同类型的key呢？**

我们可以通过给key添加前缀加以区分，不过这个前缀不是随便加的，有一定的规范：

Redis的key允许有多个单词形成层级结构，多个单词之间用':'隔开，格式如下：

```
项目名:业务名:类型:id
```

**如果Value是一个Java对象，例如一个User对象，则可以将对象序列化为JSON字符串后存储：**

| **KEY**         | **VALUE**                                  |
| --------------- | ------------------------------------------ |
| heima:user:1    | {"id":1,  "name": "Jack", "age": 21}       |
| heima:product:1 | {"id":1,  "name": "小米11", "price": 4999} |

### 实战

~~~bash
127.0.0.1:6379> set hdb:user:1 '{"id":1, "name":"Jack", "age": 21}'
OK
127.0.0.1:6379> set hdb:user:2 '{"id":2, "name":"Rose", "age": 18}'
OK
127.0.0.1:6379> set hdb:product:1 '{"id":1, "name":"小米11", "price": 4999}'
OK
127.0.0.1:6379> set hdb:product:2 '{"id":2, "name":"荣耀6", "price": 2999}'
OK
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/17/99830_image-20240117215605769.png" alt="image-20240117215605769" style="zoom:67%;" />

## hash类型

String结构是将对象序列化为JSON字符串后存储，当需要修改对象某个字段时很不方便

Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD。

> Create、Read、Update和Delete

**Hash的常见命令有：**

- HSET key field value：添加或者修改hash类型key的field的值

- HGET key field：获取一个hash类型key的field的值

- HMSET：批量添加多个hash类型key的field的值

- HMGET：批量获取多个hash类型key的field的值

- HGETALL：获取一个hash类型的key中的所有的field和value
- HKEYS：获取一个hash类型的key中的所有的field
- HINCRBY:让一个hash类型key的字段值自增并指定步长
- HSETNX：添加一个hash类型的key的field值，前提是这个field不存在，否则不执行

### HSET

~~~bash
127.0.0.1:6379> hset hdb:user:3 name lucy
(integer) 1
127.0.0.1:6379> hset hdb:user:3 age 23
(integer) 1
~~~

### HGET

~~~bash
127.0.0.1:6379> hget hdb:user:3 age
"28"
~~~

### HMGET

~~~bash
127.0.0.1:6379> hmget hdb:user:3 age name
1) "28"
2) "lucy"
~~~

### hgetall

~~~bash
127.0.0.1:6379> hgetall hdb:user:3
1) "name"
2) "lucy"
3) "age"
4) "28"
5) "sex"
6) "girl"
~~~

### hkeys

~~~bash
127.0.0.1:6379> hkeys hdb:user:3
1) "name"
2) "age"
3) "sex"
~~~

### hvals

~~~bash
127.0.0.1:6379> hvals hdb:user:3
1) "lucy"
2) "28"
3) "girl"
~~~

### hincrby

~~~bash
127.0.0.1:6379> hincrby hdb:user:3 age 2
(integer) 30
127.0.0.1:6379> hincrby hdb:user:3 age 2
(integer) 32

~~~

## List类型

特征也与LinkedList类似：

- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。

List的常见命令有：

- LPUSH key element ... ：向列表左侧插入一个或多个元素
- LPOP key：移除并返回列表左侧的第一个元素，没有则返回nil
- RPUSH key element ... ：向列表右侧插入一个或多个元素
- RPOP key：移除并返回列表右侧的第一个元素
- LRANGE key star end：返回一段角标范围内的所有元素
- BLPOP和BRPOP：（BLPOP）移除并返回列表中的第一个元素。否则会阻塞直到有元素可用。如果最后一个元素被弹出，则删除列表

~~~bash
127.0.0.1:6379> LPUSH users 1 2 3 4 5
(integer) 5
127.0.0.1:6379> RPUSH animals 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LPOP users 1
1) "5"
127.0.0.1:6379> LRANGE users 0 2
1) "4"
2) "3"
3) "2"
~~~

### BLPOP

没有deals，阻塞，等待100s

~~~bash
BLPOP deals 100
~~~

新建终端，push一个元素

~~~bash
127.0.0.1:6379> lpush deals egg
(integer) 1
~~~

查看原终端

~~~bash
127.0.0.1:6379> BLPOP deals 100
1) "deals"
2) "egg"
(49.58s)
~~~
如果最后一个元素被弹出，则删除列表
~~~bash
127.0.0.1:6379> keys deals
(empty array)
~~~

## Set类型

Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：

- 无序

- 元素不可重复

- 查找快

- 支持交集、并集、差集等功能

Set的常见命令有：

- SADD key member ... ：向set中添加一个或多个元素
- SREM key member ... : 移除set中的指定元素
- SCARD key： 返回set中元素的个数
- SISMEMBER key member：判断一个元素是否存在于set中
- SMEMBERS：获取set中的所有元素
- SINTER key1 key2 ... ：求key1与key2的交集

### SADD

~~~bash
127.0.0.1:6379> SADD s1 a b c
(integer) 3
~~~

### SMEMBERS

~~~bash
127.0.0.1:6379> SMEMBERS s1
1) "a"
2) "b"
3) "c"
~~~

### SREM

~~~bash
127.0.0.1:6379> SREM s1 a
(integer) 1
~~~

### SISMEMBER

~~~bash
127.0.0.1:6379> SISMEMBER s1 a
(integer) 0
~~~

### SCARD

~~~bash
127.0.0.1:6379> SCARD s1
(integer) 2
~~~

### 交集SINTER

数据准备

~~~bash
127.0.0.1:6379> SADD zs lisi wangwu zhouliu
(integer) 3
127.0.0.1:6379> SADD lisi wangwu mazi ergou
(integer) 3
~~~

~~~bash
127.0.0.1:6379> SINTER zs lisi
1) "wangwu"
~~~

### 差集SDIFF

~~~bash
127.0.0.1:6379> SDIFF zs lisi
1) "lisi"
2) "zhouliu"
~~~

### 并集SUNION

~~~bash
127.0.0.1:6379> SUNION zs lisi
1) "lisi"
2) "wangwu"
3) "zhouliu"
4) "mazi"
5) "ergou"
~~~

## SortedSet类型

Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

SortedSet具备下列特性：

- 可排序
- 元素不重复
- 查询速度快

因为SortedSet的可排序特性，**经常被用来实现排行榜这样的功能**。



SortedSet的常见命令有：

- ZADD key score member：添加一个或多个元素到sorted set ，如果已经存在则更新其score值
- ZREM key member：删除sorted set中的一个指定元素
- ZSCORE key member : 获取sorted set中的指定元素的score值
- ZRANK key member：获取sorted set 中的指定元素的排名
- ZCARD key：获取sorted set中的元素个数
- ZCOUNT key min max：统计score值在给定范围内的所有元素的个数
- ZINCRBY key increment member：让sorted set中的指定元素自增，步长为指定的increment值
- ZRANGE key min max：按照score排序后，获取指定排名范围内的元素
- ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素
- ZDIFF、ZINTER、ZUNION：求差集、交集、并集

注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可，例如：

- **升序**获取sorted set 中的指定元素的排名：ZRANK key member

- **降序**获取sorted set 中的指定元素的排名：ZREVRANK key memeber

### ZADD

~~~bash
127.0.0.1:6379> ZADD students 85 jack 89 lucy 82 rose 95 tom 78 jerry 92 amy 76 miles
(integer) 7
~~~

### ZREM

~~~bash
127.0.0.1:6379> ZREM students tom
(integer) 1
~~~

### ZSCORE

~~~bash
127.0.0.1:6379> zscore students amy
"92"
~~~

### ZRANK 

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/18/65612_image-20240118105909641.png" alt="image-20240118105909641" style="zoom: 80%;" />

从低到高排名

~~~bash
127.0.0.1:6379> ZRANK students amy
(integer) 5
~~~

从高到底排名

~~~bash
127.0.0.1:6379> ZREVRANK students amy
(integer) 0
~~~

### ZCARD

~~~bash
127.0.0.1:6379> ZCARD students
(integer) 6
~~~

### ZCOUNT

ZCOUNT key min max：统计score值在给定范围内的所有元素的个数

~~~bash
127.0.0.1:6379> ZCOUNT students 0 90
(integer) 5
~~~

### ZINCRBY

ZINCRBY key increment member：让sorted set中的指定元素自增，步长为指定的increment值

~~~bash
127.0.0.1:6379> ZINCRBY students 2  amy
"94"
~~~

### ZRANGE

ZRANGE key min max：按照score排序后，获取指定排名范围内的元素（升序）

~~~bash
127.0.0.1:6379> ZREVRANGE students 0 3
1) "amy"
2) "lucy"
3) "jack"
4) "rose"
~~~

### ZRANGEBYSCORE

ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素

查找80分以下的同学

~~~bash
127.0.0.1:6379> ZRANGEBYSCORE students 0 80
1) "miles"
2) "jerry"
~~~

