---
title: springboot3 场景整合
categories:
  - springboot3
tags:
  - springboot3
---
<meta name="referrer" content="no-referrer"/>

## springboot3笔记资料地址

[springboot3-notes](https://www.yuque.com/leifengyang/springboot3)

# NoSQL

## 准备工作

### 安装docker

- 准备prometheus.yml，docker-compose.yml
- 启动环境

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

## Redis整合

### 配置

```properties
spring.data.redis.host=server ip
spring.data.redis.port=6379
```

### 测试

~~~java
 /**
     * set:     集合:       redisTemplate.opsForSet()
     */
    @Test
    void testSet(){

        String setName = "settest";
        //1、给集合中添加元素
        redisTemplate.opsForSet().add(setName,"1","2","3","3");

        Boolean aBoolean = redisTemplate.opsForSet().isMember(setName, "2");
        Assertions.assertTrue(aBoolean);

        Boolean aBoolean1 = redisTemplate.opsForSet().isMember(setName, "5");
        Assertions.assertFalse(aBoolean1);
    }
~~~

## RedisTemplate定制化

模仿官方的RedisTemplate进行改写默认序列化，在`RedisAutoConfiguration`中有
找到`RedisSerializer`接口，定位实现类（ctr+H），发现`GenericJackson2JsonRedisSerializer`

~~~java
@Configuration
public class AppRedisConfiguration {

    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        //把对象转为json字符串的序列化工具
        template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
~~~

![image-20240109102411367](https://gitee.com/hollis7/pictures/raw/master/2024/01/09/78210_image-20240109102411367.png)

重新运行程序，会产生一个没有乱码的person的`<key,value>`

![image-20240109102745029](https://gitee.com/hollis7/pictures/raw/master/2024/01/09/82329_image-20240109102745029.png)

## redis相关配置

- **Redis客户端**

- - Lettuce： 默认
  - Jedis：可以使用以下切换

~~~properties
#spring.data.redis.client-type=lettuce

#设置lettuce的底层参数
spring.data.redis.lettuce.pool.enabled=true
spring.data.redis.lettuce.pool.max-active=8
~~~
使用Jedis，在redis-starter中排除`lettuce`，再导入`jedis`
~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

<!--        切换 jedis 作为操作redis的底层客户端-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
~~~

# 接口文档

Swagger 可以快速生成**实时接口**文档，方便前后开发人员进行协调沟通。遵循 **OpenAPI** 规范。

## 配置

~~~xml
<dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.1.0</version>
</dependency>
~~~

进入`http://localhost:8080/swagger-ui/index.html`

## 常用注解

| 注解         | 标注位置            | 作用                   |
| ------------ | ------------------- | ---------------------- |
| @Tag         | controller 类       | 标识 controller 作用   |
| @Parameter   | 参数                | 标识参数作用           |
| @Parameters  | 参数                | 参数多重说明           |
| @Schema      | model 层的 JavaBean | 描述模型作用及每个属性 |
| @Operation   | 方法                | 描述方法作用           |
| @ApiResponse | 方法                | 描述响应状态码等       |

## controller示例

~~~java
@Tag(name="部门",description = "部门的CRUD")
@RestController
public class DeptController {

    @Autowired
    DeptService deptService;


    //Knife4j
    @Operation(summary = "查询",description = "按照id查询部门")
    @GetMapping("/dept/{id}")
    public Dept getDept(@PathVariable("id") Long id){
        return deptService.getDeptById(id);
    }

    @Operation(summary = "查询所有部门")
    @GetMapping("/depts")
    public List<Dept> getDept(){
        return deptService.getDepts();
    }


    @Operation(summary = "保存部门",description = "必须提交json")
    @PostMapping("/dept")
    public String saveDept(@RequestBody Dept dept){
        deptService.saveDept(dept);
        return "ok";
    }

    @Operation(summary = "按照id删除部门",description = "必须提交json")
    @DeleteMapping("/dept/{id}")
    public String deleteDept(@PathVariable("id") @Parameter(description = "部门id") Long id){
        deptService.deleteDept(id);
        return "ok";
    }
}
~~~

## 分组设置

~~~java
@Configuration
public class ApiUiConfig {
    /**
     * 分组设置
     * @return
     */
    @Bean
    public GroupedOpenApi empApi() {
        return GroupedOpenApi.builder()
                .group("员工管理")
                .pathsToMatch("/emp/**","/emps")
                .build();
    }
    @Bean
    public GroupedOpenApi deptApi() {
        return GroupedOpenApi.builder()
                .group("部门管理")
                .pathsToMatch("/dept/**","/depts")
                .build();
    }

    @Bean
    public OpenAPI docsOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("SpringBoot3-CRUD API")
                        .description("专门测试接口文件")
                        .version("v0.0.1")
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")))
                .externalDocs(new ExternalDocumentation()
                        .description("哈哈 Wiki Documentation")
                        .url("https://springshop.wiki.github.org/docs"));
    }
}
~~~

# 远程调用

## 方法一：webclient

编写WeatherService

~~~java
 public String getByWebClient(String city){
        //远程调用阿里云API
        //1、创建WebClient
        WebClient client = WebClient.create();

        //2、准备数据
        Map<String,String> params = new HashMap<>();
        params.put("area",city);
        //3、定义发请求行为  CompletableFuture
        String json = client.get()
                .uri("https://qryweather.market.alicloudapi.com/lundroid/queryweather?area={area}", params)
                .accept(MediaType.APPLICATION_JSON)
                .header("Authorization", "APPCODE 9a0a550ada0c4656a5777f8ef674cbca")
                .retrieve()
                .bodyToMono(String.class)
                .block();

        return json;
    }
~~~

WeatherController调用WeatherService的getByWebClient函数

~~~java
@GetMapping("/weather")
    public String weather(@RequestParam("city")String city){
        String weather = weatherService.getByWebClient(city);
        return weather;
    }
~~~

## 方法二：HTTP Interface

Spring 允许我们通过定义接口的方式，给任意位置发送 http 请求，实现远程调用，可以用来简化 HTTP 远程访问。需要webflux场景才可

1. 定义配置类`AliyunApiConfiguration`，其中aliyun.appcode为阿里云的相关api的授权码，可在properties中设置

~~~java
@Configuration
public class AliyunApiConfiguration {
    @Bean
    HttpServiceProxyFactory httpServiceProxyFactory(@Value("${aliyun.appcode}") String appCode){
        //1、创建客户端
        WebClient client = WebClient.builder()
                .defaultHeader("Authorization","APPCODE "+appCode)
                .codecs(clientCodecConfigurer -> {
                    clientCodecConfigurer
                            .defaultCodecs()
                            .maxInMemorySize(256*1024*1024);
                    //响应数据量太大有可能会超出BufferSize，所以这里设置的大一点
                })
                .build();
        //2、创建工厂
        HttpServiceProxyFactory factory = HttpServiceProxyFactory
                .builderFor(WebClientAdapter.create(client))
                .build();
        return factory;
    }

    @Bean
    WeatherInterface weatherInterface(HttpServiceProxyFactory httpServiceProxyFactory){
        //3、获取代理对象
        WeatherInterface weatherInterface = httpServiceProxyFactory.createClient(WeatherInterface.class);
        return weatherInterface;
    }
}
~~~

~~~properties
aliyun.appcode=9a0a550ada0c4656a5777f8ef674cbca
~~~

2. 定义接口

~~~java
public interface WeatherInterface {
    @GetExchange(url = "https://qryweather.market.alicloudapi.com/lundroid/queryweather",accept = "application/json")
    String getWeather(@RequestParam("area") String city);
}
~~~

3. controller调用相关服务

~~~java
@RestController
public class WeatherController {

    @Autowired
    WeatherService weatherService;
    @GetMapping("/weather")
    public String weather(@RequestParam("city")String city){
        String weather = weatherService.weather(city);
        return weather;
    }

}
~~~

4. 当有新的阿里云api时，可以复用配置代码

~~~java
//AliyunApiConfiguration
@Bean
    ExpressApi expressApi(HttpServiceProxyFactory httpServiceProxyFactory){
        ExpressApi client = httpServiceProxyFactory.createClient(ExpressApi.class);
        return client;
    }
~~~

~~~java
//WeatherController
public interface ExpressApi {
    @GetExchange(url = "https://qryweather.market.alicloudapi.com/lundroid/queryweather",accept = "application/json")
    String getExpress(@RequestParam("number") String number);
}
~~~

# 消息服务

## kafka原理

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1683170677428-6ffa28b6-d522-435f-9e50-20fe3ddfd024.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_36%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1125%2Climit_0)

## 准备

1. 新建module，勾选

- [x] Lombok
- [x] Spring Web
- [x] Spring for Apache Kafka

![image-20240109203533200](https://gitee.com/hollis7/pictures/raw/master/2024/01/09/12022_image-20240109203533200.png)

2. 配置kafka服务器

~~~properties
spring.kafka.bootstrap-servers=server_ip:9092
~~~

3. 在测试主程序中编写程序，需要修改hosts文件，文件地址：

```
hosts：
C:\Windows\System32\drivers\etc
```
:rage: 加入ip对应的名字，不是老师讲的kafka

修改hosts文件，先以管理员权限打开文本编辑器，再用文本编辑器打开hosts文件进行修改。

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/09/86312_image-20240109213939272.png" alt="image-20240109213939272"  />

## 测试（异步）

~~~java
@Test
    void contextLoads() {
        StopWatch stopWatch = new StopWatch();
        CompletableFuture[] futures = new CompletableFuture[10000];
        stopWatch.start();
        for (int i = 0; i < 10000; i++) {
            CompletableFuture future = kafkaTemplate.send("news", "pig", "你是猪");
            futures[i] = future;
        }
        CompletableFuture.allOf(futures).join();
        stopWatch.stop();
        
        long millis = stopWatch.getTotalTimeMillis();
        System.out.println("10000消息发送完成：ms时间：" + millis);
    }
~~~

## kafka存入类测试

~~~java
@Test
    void send() {
        CompletableFuture future = kafkaTemplate.send("news", "person", new Person(1L, "张三", "hjaha@qq.com"));
        future.join();
        System.out.println("消息发送成功...");
    }
~~~

需要进行序列化配置，默认是`Serializer`，改为json

~~~properties
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
~~~

## KafkaListener监听

~~~java
@Component
public class MyHahaTopicListener {
    //默认的监听是从消息队列最后一个消息开始拿。新消息才能拿到
    @KafkaListener(topics = "news", groupId = "haha")
    public void listenRecords(ConsumerRecord record) {
        Object key = record.key();
        Object value = record.value();
        System.out.println("收到消息：key【" + key + "】 value【" + value + "】");
    }

    //拿到以前的完整消息;
    @KafkaListener(groupId = "haha2",
            topicPartitions = {
                    @TopicPartition(topic = "news",
                            partitionOffsets = {
                                    @PartitionOffset(partition = "0", initialOffset = "0")
                            })
            })
    public void listenAllRecords(ConsumerRecord record) {
        Object key = record.key();
        Object value = record.value();
        System.out.println("=============收到消息：key【" + key + "】 value【" + value + "】");
    }
}
~~~

~~~java
@EnableKafka
@SpringBootApplication
public class Boo312MessageApplication
~~~

Kafka 的特点包括：

- 高吞吐：Kafka 可以处理每秒数百万条消息。
- 低延迟：Kafka 可以将消息从生产者发送到消费者的延迟保持在毫秒级。
- 持久化：Kafka 将消息持久化到磁盘，即使生产者或消费者发生故障，消息也不会丢失。
- 扩展性：Kafka 可以水平扩展，以满足不断增长的需求。
