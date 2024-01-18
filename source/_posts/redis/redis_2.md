---
title: redis_2
categories:
- redis
tags:
- redis
- java
---
<meta name="referrer" content="no-referrer"/>

## 内容包括

- redis java 客户端
- SpringDataRedis
- 自定义序列化
- 手动序列化

<!--more-->

## redis的java客户端

在Redis官网中提供了各种语言的客户端，地址：https://redis.io/docs/clients/

## Jedis客户端

Jedis的官网地址： https://github.com/redis/jedis

略，重点学习SpringDataRedis

## SpringDataRedis

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis，官网地址：https://spring.io/projects/spring-data-redis

* 提供了对不同Redis客户端的整合（Lettuce和Jedis）
* 提供了RedisTemplate统一API来操作Redis
* 支持Redis的发布订阅模型
* 支持Redis哨兵和Redis集群
* 支持基于Lettuce的响应式编程
* 支持基于JDK.JSON.字符串.Spring对象的数据序列化及反序列化
* 支持基于Redis的JDKCollection实现

SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

![1652976773295](https://gitee.com/hollis7/pictures/raw/master/2024/01/18/24230_1652976773295.png)

## SpringDataRedis项目创建

![image-20240118152046621](https://gitee.com/hollis7/pictures/raw/master/2024/01/18/43447_image-20240118152046621.png)

1、加入依赖

~~~xml
<!--common-pool-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
~~~

2、配置yaml文件

~~~yaml
spring:
  data:
    redis:
      host: ip
      port: 6379
      password: xxxxx
      lettuce:
        pool:
          max-active: 8  #最大连接
          max-idle: 8   #最大空闲连接
          min-idle: 0   #最小空闲连接
          max-wait: 100ms #连接等待时间
~~~

3、RedisTemplate测试

~~~java
@Test
void contextLoads() {
    redisTemplate.opsForValue().set("name", "彬哥");
    String name = (String) redisTemplate.opsForValue().get("name");
    System.out.println("name = " + name);
}
~~~

4、发现问题

~~~bash
127.0.0.1:6379> get name
(nil)
~~~

~~~bash
127.0.0.1:6379> keys *
 1) "hdb:product:2"
 5) "users"
 6) "\xac\xed\x00\x05t\x00\x04name"
~~~

### 数据序列化器

RedisTemplate可以接收任意Object作为值写入Redis

只不过写入前会把Object序列化为字节形式，**默认是采用JDK序列化**

### 自定义RedisTemplate的序列化

~~~java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =
                new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return template;
    }
}
~~~

## StringRedisTemplate

尽管JSON的序列化方式可以满足我们的需求，但依然存在一些问题，如图：

![image-20240118194242562](https://gitee.com/hollis7/pictures/raw/master/2024/01/18/57696_image-20240118194242562.png)

-  `"@class": "com.hdb.redisdemo.pojo.User",`占用的空间比对象本身还多，但是自动反序列化又需要。
- 为了减少内存的消耗，我们可以采用手动序列化的方式，换句话说，就是不借助默认的序列化器，而是我们自己来控制序列化的动作，同时，我们只采用String的序列化器，这样，在存储value时，我们就不需要在内存中就不用多存储数据，从而节约我们的内存空间
- SpringDataRedis就提供了RedisTemplate的子类：StringRedisTemplate，它的key和value的序列化方式默认就是String方式。

~~~java
@SpringBootTest
public class RedisStringTests {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testSaveUser() throws JsonProcessingException {
        // 创建对象
        User user = new User("bin", 21);
        // 手动序列化
        String json = mapper.writeValueAsString(user);
        // 写入数据
        stringRedisTemplate.opsForValue().set("user:200", json);
        // 获取数据
        String jsonUser = stringRedisTemplate.opsForValue().get("user:200");
        // 手动反序列化
        User user1 = mapper.readValue(jsonUser, User.class);
        System.out.println("user1 = " + user1);

    }


}
~~~

## 存储hash 

~~~java
@Test
void testHash() {
    stringRedisTemplate.opsForHash().put("user:400", "name", "bin");
    stringRedisTemplate.opsForHash().put("user:400", "age", "21");

    Map<Object, Object> entries = stringRedisTemplate.opsForHash().entries("user:400");
    System.out.println("entries = " + entries);
}
~~~