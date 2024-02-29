---
title: redis_实战篇_商户查询缓存
categories:
- redis
tags:
- redis
- java
---
<meta name="referrer" content="no-referrer"/>

## 内容

- 缓存
- 商铺查询（单一）
- 商铺类型查询（List）
- 缓存穿透
- 数据库和缓存不一致
- 缓存穿透
- 缓存雪崩
- 缓存击穿
- 封装redis工具类

<!--more-->

## 缓存

**缓存(**Cache),就是数据交换的**缓冲区**,俗称的缓存就是**缓冲区内的数据**,一般从数据库中获取,存储于本地代码。

### 作用

- 降低后端负载
- 提高读写效率，降低响应时间

### 缓存的成本

- 数据一致性成本
- 代码维护成本
- 运维成本

# 商户查询缓存

## 流程图

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/27/68828_1653322097736.png" alt="1653322097736" style="zoom: 45%;" />

## 商铺查询

### 步骤

1. 从redis查询商铺缓存
2. 判断是否存在
3. 存在，直接返回
4. 不存在，根据1d查询数据库
5. 不存在，返回错误
6. 存在，写入redis

### 代码

~~~java
@Service
public class ShopServiceImpl extends ServiceImpl<ShopMapper, Shop> implements IShopService {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result queryById(Long id) {
        //CACHE_SHOP_KEY--->"cache:shop:"
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查询商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
         //2.判断是否存在
        if (StrUtil.isNotBlank(shopJson)) {
            //3.存在，直接返回
            Shop shop = JSONUtil.toBean(shopJson, Shop.class);
            return Result.ok(shop);
        }
        //4.不存在，根据1d查询数据库
        Shop shop = getById(id);
        //5.不存在，返回错误
        if (shop == null) {
            return Result.fail("店铺不存在！");
        }
        //6.存在，写入redis
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop));
        //7.返回
        return Result.ok(shop);
    }
}
~~~

## 作业 shoptype缓存

### controller源码

~~~java
 @GetMapping("list")
    public Result queryTypeList() {
        List<ShopType> typeList = typeService
                .query().orderByAsc("sort").list();
        return Result.ok(typeList);
    }
~~~

### 实现类

~~~java
@Service
public class ShopTypeServiceImpl extends ServiceImpl<ShopTypeMapper, ShopType> implements IShopTypeService {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result queryByRedis() {
        String shopTypeListJson = stringRedisTemplate.opsForValue().get(CACHE_SHOP_TYPE_LIST);

        if (StrUtil.isNotBlank(shopTypeListJson)) {
            List<ShopType> shopTypeLists = JSONUtil.toList(shopTypeListJson, ShopType.class);
            return Result.ok(shopTypeLists);
        }

        List<ShopType> shopTypeLists = query().orderByAsc("sort").list();
        if (shopTypeLists.isEmpty()) {
            return Result.fail("没有店铺类型");
        }
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_TYPE_LIST, JSONUtil.toJsonStr(shopTypeLists));
        return Result.ok(shopTypeLists);
    }
}
~~~

## 缓存更新策略

缓存更新是redis为了节约内存而设计出来的一个东西，主要是因为内存数据宝贵，当我们向redis插入太多数据，此时就可能会导致缓存中的数据过多，所以redis会对部分数据进行更新，或者把他叫为淘汰更合适。

**内存淘汰：**redis自动进行，当redis内存达到咱们设定的max-memery的时候，会自动触发淘汰机制，淘汰掉一些不重要的数据(可以自己设置策略方式)

**超时剔除：**当我们给redis设置了过期时间ttl之后，redis会将超时的数据进行删除，方便咱们继续使用缓存

**主动更新：**我们可以手动调用方法把缓存删掉，通常用于解决缓存和数据库不一致问题

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/28/80353_1653322506393.png" alt="1653322506393" style="zoom:40%;" />

## 数据库和缓存不一致采用什么方案

**Cache Aside Pattern 人工编码方式**：缓存调用者在更新完数据库后再去更新缓存，也称之为双写方案

* 删除缓存还是更新缓存？

  * 更新缓存：每次更新数据库都更新缓存，无效写操作较多
  * 删除缓存：更新数据库时让缓存失效，查询时再更新缓存

* 如何保证缓存与数据库的操作的同时成功或失败？

  * 单体系统，将缓存与数据库操作放在一个事务
  * 分布式系统，利用TCC等分布式事务方案

- 先操作缓存还是先操作数据库？

  * 先删除缓存，再操作数据库

  * 先操作数据库，再删除缓存


![1653323595206](C:\data\mysoftware\Typora\typoraPicture\1653323595206.png)

更新数据库业务耗时更长，查询和写入redis缓存可能在更新数据库前完成

写入缓存时间短，再次期间更新数据库和删除缓存的可能性小

### 代码修改

**设置超时**

~~~java
stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
~~~

**updata代码**：先操作数据库，再删除缓存

`ShopServiceImpl`

~~~java
@Override
    @Transactional
    public Result update(Shop shop) {
        Long id = shop.getId();
        if (id == null) {
            return Result.fail("店铺id不能为空！");
        }
        // 1.更新数据库
        updateById(shop);
        // 2.删除缓存
        stringRedisTemplate.delete(CACHE_SHOP_KEY + shop.getId());
        return Result.ok();
    }
~~~

这段代码中的 `@Transactional` 注解用于标识这个方法应该在一个事务中执行。事务是数据库操作的一种机制，它确保了一组数据库操作要么全部成功执行，要么全部失败回滚，以保持数据的一致性和完整性。

## 缓存穿透

缓存穿透 ：缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。

常见的解决方案有两种：

* 缓存空对象
  * 优点：实现简单，维护方便
  * 缺点：
    * 额外的内存消耗
    * 可能造成短期的不一致
* 布隆过滤
  * 优点：内存占用较少，没有多余key
  * 缺点：
    * 实现复杂
    * 存在误判可能

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/28/94607_1653326156516.png" alt="1653326156516" style="zoom: 67%;" />

### 代码实现-缓存空对象

![1653327124561](https://gitee.com/hollis7/pictures/raw/master/2024/02/28/77659_1653327124561.png)

~~~java
 @Override
    public Result queryById(Long id) {
        //CACHE_SHOP_KEY--->"cache:shop:"
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查询商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isNotBlank(shopJson)) {
            //3.存在，直接返回
            Shop shop = JSONUtil.toBean(shopJson, Shop.class);
            return Result.ok(shop);
        }
        //空字符串
        if (shopJson != null) {
            //返回错误信息
            return Result.fail("店铺不存在！");
        }
        //4.不存在，根据1d查询数据库
        Shop shop = getById(id);
        //5.不存在，返回错误
        if (shop == null) {
            //将空值写入redis
            stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return Result.fail("店铺不存在！");
        }
        //6.存在，写入redis
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
        //7.返回
        return Result.ok(shop);
    }
~~~

## 缓存雪崩

缓存雪崩是指在同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。

解决方案：

* **给不同的Key的TTL添加随机值**
* 利用Redis集群提高服务的可用性
* 给缓存业务添加降级限流策略
* 给业务添加多级缓存

## 缓存击穿

缓存击穿问题也叫热点Key问题，就是一个被**高并发访问**并且**缓存重建业务较复杂**的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

常见的解决方案有两种：

* 互斥锁
* 逻辑过期

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/28/96191_1653328288627.png" alt="1653328288627" style="zoom:67%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/28/59776_1653328663897.png" alt="1653328663897" style="zoom:67%;" />

> 我们把过期时间设置在 redis的value中，注意：这个过期时间并不会直接作用于redis，而是我们后续通过逻辑去处理。假设线程1去查询缓存，然后从value中判断出来当前的数据已经过期了，此时线程1去获得互斥锁，那么其他线程会进行阻塞，获得了锁的线程他会开启一个 线程去进行 以前的重构数据的逻辑，直到新开的线程完成这个逻辑后，才释放锁， 而线程1直接进行返回，假设现在线程3过来访问，由于线程线程2持有着锁，所以线程3无法获得锁，线程3也直接返回数据，只有等到新开的线程2把重建数据构建完后，其他线程才能走返回正确的数据。
>
> 这种方案巧妙在于，异步的构建缓存，缺点在于在构建完缓存之前，返回的都是脏数据。

### 互斥锁

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/28/88931_1653357860001.png" alt="1653357860001" style="zoom: 50%;" />

#### 代码

`ShopServiceImpl`

isLock那里一定要return

~~~java
 @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result queryById(Long id) {
        Shop shop = queryWithMutex(id);
        if (shop == null) {
            Result.fail("店铺不存在！！！");
        }
        return Result.ok(shop);
    }

    public Shop queryWithMutex(Long id) {
        //CACHE_SHOP_KEY--->"cache:shop:"
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查询商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isNotBlank(shopJson)) {
            //3.存在，直接返回
            return JSONUtil.toBean(shopJson, Shop.class);
        }
        //空字符串
        if (shopJson != null) {
            //返回错误信息
            return null;
        }
        // 4.实现缓存重构
        //4.1 获取互斥锁
        String lockKey = "lock:shop:" + id;
        Shop shop = null;
        try {
            boolean isLock = tryLock(lockKey);
            // 4.2 判断否获取成功
            if (!isLock) {
                //4.3 失败，则休眠重试
                Thread.sleep(50);
                //这里一定要return，否则会多次查询数据库
                return queryWithMutex(id);
            }
            //4.4 成功，根据id查询数据库
            shop = getById(id);

            //模拟重建延迟
            Thread.sleep(200);
            //5.不存在，返回错误
            if (shop == null) {
                //将空值写入redis
                stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
                return null;
            }
            //6.存在，写入redis
            stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            //7.释放互斥锁
            unLock(lockKey);
        }
        //8.返回
        return shop;
    }

    @Override
    @Transactional
    public Result update(Shop shop) {
        Long id = shop.getId();
        if (id == null) {
            return Result.fail("店铺id不能为空！");
        }
        // 1.更新数据库
        updateById(shop);
        // 2.删除缓存
        stringRedisTemplate.delete(CACHE_SHOP_KEY + shop.getId());
        return Result.ok();
    }

    private boolean tryLock(String key) {
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
        return BooleanUtil.isTrue(flag);
    }

    private void unLock(String key) {
        stringRedisTemplate.delete(key);
    }
~~~

### 逻辑过期

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/02/29/21339_1653360308731.png" alt="1653360308731" style="zoom: 55%;" />

思路分析：当用户开始查询redis时，判断是否命中，如果没有命中则直接返回空数据，不查询数据库，而一旦命中后，将value取出，判断value中的过期时间是否满足，如果没有过期，则直接返回redis中的数据，如果过期，则在开启独立线程后直接返回之前的数据，独立线程去重构数据，重构完成后释放互斥锁。

#### 代码

**封装过期时间和店铺数据**

~~~java
private void saveShop2Redis(Long id, Long expireSeconds) {
        // 1.查询商铺数据
        Shop shop = getById(id);
        //2.封装逻辑过期时间
        RedisData redisData = new RedisData();
        redisData.setData(shop);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireSeconds));
        //3.写入redis
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(redisData));
    }
~~~

**使用逻辑过期解决缓存击穿**

~~~java
public Shop queryWithLogicalExpire(Long id) {
        //CACHE_SHOP_KEY--->"cache:shop:"
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查询商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isBlank(shopJson)) {
            //3.未命中
            return null;
        }
        // 4.命中，需要先把json反序列化为对象
        RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
        Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);
        LocalDateTime expireTime = redisData.getExpireTime();
        // 5.判断是否过期
        if (expireTime.isAfter(LocalDateTime.now())) {
            // 5.1.未过期，直接返回店铺信息
            return shop;
        }
        // 5.2.已过期，需要缓存重建
        // 6.缓存重建
        // 6.1.获取互斥锁
        String lockKey = LOCK_SHOP_KEY + id;
        boolean isLock = tryLock(lockKey);
        // 6.2.判断是否获取锁成功
        if (isLock) {
            CACHE_REBUILD_EXECUTOR.submit(() -> {
                try {
                    //重建缓存
                    this.saveShop2Redis(id, 20L);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                } finally {
                    //释放锁
                    unLock(lockKey);
                }
            });
        }
        //8.返回
        return shop;
    }
~~~

### 封装redis工具类

善于利用泛型，函数式编程

R是返回类型，ID不确定类型，不知道具体的数据库搜索函数，采用函数式编程

~~~java
public <R, ID> R queryWithPassThrough(
            String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
        String key = keyPrefix + id;
        // 1.从redis查询商铺缓存
        String json = stringRedisTemplate.opsForValue().get(key);
        // 2.判断是否存在
        if (StrUtil.isNotBlank(json)) {
            // 3.存在，直接返回
            return JSONUtil.toBean(json, type);
        }
        // 判断命中的是否是空值
        if (json != null) {
            // 返回一个错误信息
            return null;
        }

        // 4.不存在，根据id查询数据库
        R r = dbFallback.apply(id);
        // 5.不存在，返回错误
        if (r == null) {
            // 将空值写入redis
            stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            // 返回错误信息
            return null;
        }
        // 6.存在，写入redis
        this.set(key, r, time, unit);
        return r;
    }
~~~

调用封装类

~~~java
Shop shop = cacheClient
                .queryWithPassThrough(CACHE_SHOP_KEY, id, Shop.class, this::getById, CACHE_SHOP_TTL, TimeUnit.MINUTES);
~~~

或者

~~~java
Shop shop = cacheClient
    .queryWithPassThrough(CACHE_SHOP_KEY, id, Shop.class, id2 -> getById(id2), CACHE_SHOP_TTL, TimeUnit.MINUTES);
~~~

