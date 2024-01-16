---
title: springboot3 web开发
categories:
  - springboot3
tags:
  - springboot3
---
<meta name="referrer" content="no-referrer"/>

## springboot3笔记资料地址

[springboot3-notes](https://www.yuque.com/leifengyang/springboot3)

## xmind文件打开网址

[幕布](https://mubu.com/app/edit/home/1IVQRB_zQ2b#m)

## 关闭占用端口的程序

查看占用端口的程序

~~~bash
netstat -ano | findstr 8080
~~~

使用命令关闭

~~~bash
taskkill -PID 进程号 -F
~~~

## springboot3 识别不了新module

### 法一

1. Remove Module
2. 选中未识别的module

![image-20231130103414289](https://gitee.com/hollis7/pictures/raw/master/2023/11/30/60877_image-20231130103414289.png)

3. `import module from external model---->maven`

### 法二

1. 右击 项目名，选择“ Add Framework Support”
2. 选中`maven`

## SpringBoot

SpringBoot 帮我们简单、快速地创建一个独立的、生产级别的 **Spring 应用**

**特性：**

- 快速创建独立 Spring 应用

- - SSM：导包、写配置、启动运行

- 直接嵌入Tomcat、Jetty or Undertow（无需部署 war 包）【Servlet容器】

- - linux  java tomcat mysql： war 放到 tomcat 的 webapps下
  - jar： java环境；  java -jar

- **重点**：提供可选的starter，简化应用**整合**

- - **场景启动器**（starter）：web、json、邮件、oss（对象存储）、异步、定时任务、缓存...
  - 导包一堆，控制好版本。
  - 为每一种场景准备了一个依赖； **web-starter。mybatis-starter**

- **重点：**按需自动配置 Spring 以及 第三方库

- - 如果这些场景我要使用（生效）。这个场景的所有配置都会自动配置好。
  - **约定大于配置**：每个场景都有很多默认配置。
  - 自定义：配置文件中修改几项就可以

- 提供生产级特性：如 监控指标、健康检查、外部化配置等

- - 监控指标、健康检查（k8s）、外部化配置

- 无代码生成、无xml

## 快速入门

1. 创建新module

![image-20231122105332075](https://gitee.com/hollis7/pictures/raw/master/2023/11/22/31077_image-20231122105332075.png)

2. 继承spring-boot-starter-parent，pom文件导入，刷新

~~~xml
 <!--    所有springboot项目都必须继承自 spring-boot-starter-parent -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.5</version>
    </parent>
~~~

3. 添加依赖

   ~~~xml
    <dependencies>
           <!--        web开发的场景启动器 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
       </dependencies>
   ~~~

![image-20231122110019745](https://gitee.com/hollis7/pictures/raw/master/2023/11/22/69206_image-20231122110019745.png)

4. 编写代码

主程序

~~~java
@SpringBootApplication //这是一个SpringBoot应用
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
~~~

5. 编写controller

~~~java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){

        return "Hello,Spring Boot 3!";
    }
}
~~~

6. 导入打包插件，并清空和打包

~~~xml
<!--    SpringBoot应用打包插件-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/22/64700_image-20231122111309675.png" alt="image-20231122111309675" style="zoom:50%;" />

7. 找到打包文件，cmd命令输入`java -jar demo.jar`运行

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/22/33036_image-20231122112346584.png" alt="image-20231122112346584" style="zoom:50%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/22/13158_image-20231122112612576.png" alt="image-20231122112612576" style="zoom: 33%;" />

8. 修改端口8888，添加配置文件`application.properties`(模拟linux场景)

![image-20231122113212064](https://gitee.com/hollis7/pictures/raw/master/2023/11/22/67131_image-20231122113212064.png)

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20231122113233472.png" alt="image-20231122113233472" style="zoom:50%;" />

端口变为8888

> windows场景下，idea的resource资源里添加`application.properties`（集中配置）

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/23/82336_image-20231123095704294.png" alt="image-20231123095704294" style="zoom:50%;" />

## Spring Initializr 创建向导

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/23/21135_image-20231123112601446.png" alt="image-20231123112601446" style="zoom:50%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/23/77224_image-20231123100919524.png" alt="image-20231123100919524" style="zoom:50%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/23/20035_image-20231123100938497.png" alt="image-20231123100938497" style="zoom:50%;" />

## 自定义依赖版本号

- 利用maven的就近原则

- - 直接在**导入依赖的时候声明版本**

~~~xml
<dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>${mysql.version}</version>
<!--            <version>8.0.31</version>-->
            <exclusions>
                <exclusion>
                    <groupId>com.google.protobuf</groupId>
                    <artifactId>protobuf-java</artifactId>
                </exclusion>
            </exclusions>
 </dependency>
~~~

第三方的jar包

- boot父项目没有管理的需要自行声明好

~~~xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.20</version>
</dependency>
~~~

## springboot的自动配置机制

1. springboot导入场景，容器中就会自动配置好这个场景的核心组件。

~~~java
public static void main(String[] args) {
        ConfigurableApplicationContext ioc = SpringApplication.run(Boot302DemoApplication.class, args);
        String[] names = ioc.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println("name = " + name);
        }
~~~

2. 默认的包扫描规则

- `@SpringBootApplication` 标注的类就是主程序类
- **SpringBoot只会扫描主程序所在的包及其下面的子包，自动的component-scan功能**

-  @SpringBootApplication(scanBasePackages = "com.hdb")

- `@ComponentScan("com.hdb")` 直接指定扫描的路径

3. 配置默认值

- **配置文件**的所有配置项是和某个**类的对象**值进行一一绑定的。
- 绑定了配置文件中每一项值的类： **属性类**。

- **按需加载自动配置**

> 导入场景`spring-boot-starter-web`，场景启动器除了会导入相关功能依赖，导入一个`spring-boot-starter`，是所有`starter`的`starter`，基础核心starter。`spring-boot-starter`导入了一个包 `spring-boot-autoconfigure`。包里面都是各种场景的`AutoConfiguration`**自动配置类**虽然全场景的自动配置都在 `spring-boot-autoconfigure`这个包，但是不是全都开启的。

## 常用注解

### @Bean

组件默认是单实例的，使用`@Scope("prototype")`表面多实例

~~~java
 @Scope("prototype")
    @Bean("userhaha")//替代以前的Bean标签。 组件在容器中的名字默认是方法名，可以直接修改注解的值
    public User user(){
        User user = new User();
        user.setId(1L);
        user.setName("李四");
        return user;
    }
~~~

SpringBootConfiguration和Configuration差别不大，为了便于区分

### @import

~~~java
@Import(FastsqlException.class)
~~~

使用@Import 导入第三方的组件,给容器中放指定类型的组件，组件的名字默认是全类名

## 条件注解

**@ConditionalOnClass：如果类路径中存在这个类，则触发指定行为**

**@ConditionalOnMissingClass：如果类路径中不存在这个类，则触发指定行为**

**@ConditionalOnBean：如果容器中存在这个Bean（组件），则触发指定行为**

**@ConditionalOnMissingBean：如果容器中不存在这个Bean（组件），则触发指定行为**

## @ConfigurationProperties

 **声明组件的属性和配置文件哪些前缀开始项进行绑定**

1. @ConfigurationProperties(prefix = "pig")使得通过配置文件可以给实列赋值（在Appconfig和Pig.java中均可加入）

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/24/13940_image-20231124110420990.png" alt="image-20231124110420990" style="zoom:50%;" />

~~~java
//Appconfig.class 
@Bean
    public Pig pig(){
        return new Pig();
    }
~~~

2. **@EnableConfigurationProperties：快速注册注解：**在配置文件中标注

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/24/21048_image-20231124111538865.png" alt="image-20231124111538865" style="zoom:80%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/24/15073_image-20231124111604817.png" alt="image-20231124111604817" style="zoom:50%;" />

## 中文乱码

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/24/51687_image-20231124105349873.png" alt="image-20231124105349873" style="zoom: 33%;" />

## 自动setter/getter-->@Data

pom.xml中导入依赖

~~~xml
<!--        自动生成构造器、getter/setter、自动生成Builder模式等-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>compile</scope>
        </dependency>
~~~

类中加入注释

~~~java
@Data
public class Dog {
    private Integer age;
    private String name;
}
~~~

## properties的复杂对象标识

类

~~~java
@Component
@ConfigurationProperties(prefix = "person")
@Data //自动生成JavaBean属性的getter/setter
//@NoArgsConstructor //自动生成无参构造器
//@AllArgsConstructor //自动生成全参构造器
public class Person {
    private String name;
    private Integer age;
    private Date birthDay;
    private Boolean like;
    private Child child; //嵌套对象
    private List<Dog> dogs; //数组（里面是对象）
    private Map<String,Cat> cats; //表示Map
}
~~~

配置文件`application.properties`

~~~properties
person.name=张三
person.age=18
person.birthDay=2010/10/12 12:12:12
person.like=true
person.child.name=李四
person.child.age=12
person.child.birthDay=2018/10/12
person.child.text[0]=abc
person.child.text[1]=def
person.dogs[0].name=小黑
person.dogs[0].age=3
person.dogs[1].name=小白
person.dogs[1].age=2
person.cats.c1.name=小蓝
person.cats.c1.age=3
person.cats.c2.name=小灰
person.cats.c2.age=2
~~~

配置文件二：`application.yaml`

~~~yaml
person:
  name: 张三
  age: 18
  birthDay: 2010/10/10 12:12:12
  like: true
  child:
    name: 李四
    age: 20
    birthDay: 2018/10/10
#    数组的两种表示
    text: ["abc","def"]
  dogs:
    - name: 小黑
      age: 3
    - name: 小白
      age: 2
#  map的两种表示
  cats:
    c1:
      name: 小蓝
      age: 3
    c2: {name: 小绿,age: 2} #对象也可用{}表示
~~~

## 日志配置

### 简介

以后每次勾选lombok，自动设置是set/get方法

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/30/22457_image-20231130100501188.png" alt="image-20231130100501188" style="zoom:50%;" />

**SpringBoot怎么把日志默认配置好的**

1、每个`starter`场景，都会导入一个核心场景`spring-boot-starter`

2、核心场景引入了日志的所用功能`spring-boot-starter-logging`

3、默认使用了`logback + slf4j` 组合作为默认底层日志

4、`日志是系统一启动就要用`，`xxxAutoConfiguration`是系统启动好了以后放好的组件，后来用的。

5、日志是利用**监听器机制**配置好的。`ApplicationListener`。

6、日志所有的配置都可以通过修改配置文件实现。以`logging`开始的所有配置。

![image](https://gitee.com/hollis7/pictures/raw/master/2023/11/30/68896_image.png)



~~~sh
2023-11-30T10:30:46.701+08:00  INFO 7012 --- [           main] o.a.catalina.core.AprLifecycleListener   : OpenSSL successfully initialized [OpenSSL 1.1.1v  1 Aug 2023]
2023-11-30T10:30:46.707+08:00  INFO 7012 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
~~~

### 修改日志格式

~~~properties
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} ===> %msg%n
#2023-11-30 10:45:00.046 INFO  [main] o.s.b.w.e.t.TomcatWebServer ===> Tomcat initialized with port 8080 (http)
~~~

只修改时间

~~~properties
logging.pattern.dateformat=yyyy-MM-dd HH:mm:ss
# 2023-11-30 10:51:33  INFO 20376 --- [           main] o.a.catalina.core.AprLifecycleListener   : OpenSSL successfully initialized [OpenSSL 1.1.1v  1 Aug 2023]
~~~

### 插入日志

#### 通过日志类

~~~java
@RestController
public class HelloController {
    Logger logger = LoggerFactory.getLogger(getClass());
    @GetMapping("/hello")
    public String hello(){
        logger.info("哈哈哈");
        return "hello";

    }
}
~~~

#### 注解

使用`@Slf4j`和默认的log

~~~java
@Slf4j
@RestController
public class HelloController {
    Logger logger = LoggerFactory.getLogger(getClass());
    @GetMapping("/hello")
    public String hello(){
        log.info("嘿嘿嘿");
//        logger.info("哈哈哈");
        return "hello";

    }
}
~~~

### 日志级别

- 由低到高：`ALL,TRACE, DEBUG, INFO, WARN, ERROR,FATAL,OFF`；

- - **只会打印指定级别及以上级别的日志**
  - ALL：打印所有日志
  - TRACE：追踪框架详细流程日志，一般不使用
  - DEBUG：开发调试细节日志
  - INFO：关键、感兴趣信息日志
  - WARN：警告但不是错误的信息日志，比如：版本过时
  - ERROR：业务错误日志，比如出现各种异常
  - FATAL：致命错误日志，比如jvm系统崩溃
  - OFF：**关闭所有日志记录**

#### 修改日志默认级别

~~~properties
#默认所有日志没有精确指定级别就使用root的默认级别(info)
logging.level.root=debug

#精确调整某个包下的日志级别
logging.level.com.atguigu.logging.controller=warn
~~~

#### 日志分组

~~~properties
logging.group.abc=com.atguigu.logging.controller,com.atguigu.logging.service,com.aaa,com.bbb
logging.level.abc=debug

logging.level.sql=debug
~~~

#### 指定日志文件的路径

```properties
logging.file.path=
```

#### 指定日志文件的名

指定日志文件的名： **filename 和 path的配置同时存在只看filename**

1、只写名字： 就生成到当前项目同位置的 my.log

2、**写名字+路径：生成到指定位置的指定文件**

~~~properties
logging.file.name=demo.log
~~~

## 文件归档与滚动切割

归档：每天的日志单独存到一个文档中。

切割：每个文件10MB，超过大小切割成另外一个文件。

~~~properties
#日志归档、切割
logging.logback.rollingpolicy.file-name-pattern=${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz
#存档前，每个日志文件的最大大小
logging.logback.rollingpolicy.max-file-size=1MB
~~~

上述皆为默认

## 自定义日志

`resource`下配置`logback-spring.xml`，会自动识别

# web开发

## 最佳实践

SpringBoot 已经默认配置好了**Web开发**场景常用功能。我们直接使用即可。

| 方式         | 用法                                                         | 效果                             |                                                           |
| ------------ | ------------------------------------------------------------ | -------------------------------- | --------------------------------------------------------- |
| **全自动**   | 直接编写控制器逻辑                                           |                                  | 全部使用**自动配置默认效果**                              |
| **手自一体** | `@Configuration` +   配置`**WebMvcConfigurer**`+ *配置 WebMvcRegistrations* | **不要标注** `@**EnableWebMvc**` | **保留自动配置效果** **手动设置部分功能** 定义MVC底层组件 |
| **全手动**   | `@Configuration` +   配置`**WebMvcConfigurer**`              | **标注** `@**EnableWebMvc**`     | **禁用自动配置效果** **全手动设置**                       |

## 静态资源

### 静态资源映射

静态资源映射规则在 WebMvcAutoConfiguration 中进行了定义：

1. `/webjars/**` 的所有路径 资源都在 `classpath:/META-INF/resources/webjars/`
2. `/**` 的所有路径 资源都在 `classpath:/META-INF/resources/`、`classpath:/resources/`、`classpath:/static/`、`classpath:/public/`

> 所有静态资源都定义了缓存规则。【浏览器访问过一次，就会缓存一段时间】，但此功能参数无默认值

###  Favicon

在静态资源目录下找 favicon.ico，放入static目录下，网页自动变成对应ico，记住`ctr + F5`刷新缓存

### 缓存实验

~~~properties
#开启静态资源映射规则
spring.web.resources.add-mappings=true

#设置缓存
#spring.web.resources.cache.period=3600
##缓存详细合并项控制，覆盖period配置：
## 浏览器第一次请求服务器，服务器告诉浏览器此资源缓存7200秒，7200秒以内的所有此资源访问不用发给服务器请求，7200秒以后发请求给服务器
spring.web.resources.cache.cachecontrol.max-age=7200
~~~

### 静态资源自定义配置

~~~properties
### 共享缓存
spring.web.resources.cache.cachecontrol.cache-public=true
#自定义静态资源文件夹位置
spring.web.resources.static-locations=classpath:/a/,classpath:/b/,classpath:/static/
### 2.1. 自定义webjars路径前缀
spring.mvc.webjars-path-pattern=/wj/**
### 2.2. 静态资源访问路径前缀
spring.mvc.static-path-pattern=/static/**
~~~

### 代码方式静态资源自定义配置

~~~java
@Configuration
public class MyConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //保留以前规则
        WebMvcConfigurer.super.addResourceHandlers(registry);

        //自己写新的规则。
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/a/","classpath:/b/")
                .setCacheControl(CacheControl.maxAge(1180, TimeUnit.SECONDS));
    }
}
~~~

> 自己的规则，保留`/static/下`访问，同时支持自定义下`/a`、`/b`的访问，浏览器直接输入文件名，或者`static/3.jpg`

![image-20231214203646707](https://gitee.com/hollis7/pictures/raw/master/2023/12/14/37416_image-20231214203646707.png)

#### 为什么容器中放一个WebMvcConfigurer就能配置底层行为

1. WebMvcAutoConfiguration 是一个自动配置类，它里面有一个 `EnableWebMvcConfiguration`
2. `EnableWebMvcConfiguration`继承与 `DelegatingWebMvcConfiguration`，这两个都生效
3. `DelegatingWebMvcConfiguration`利用 DI 把容器中 所有 `WebMvcConfigurer `注入进来
4. 别人调用 ``DelegatingWebMvcConfiguration`` 的方法配置底层规则，而它调用所有 `WebMvcConfigurer`的配置底层方法。

## PathPatternParser风格路径

- 默认使用新版 `PathPatternParser`进行路径匹配*，*不能匹配 `**` 在中间的情况，剩下的和 `antPathMatcher`语法兼容

- 中间有`**` ，报错，`Fix this pattern in your application or switch to the legacy parser implementation with 'spring.mvc.pathmatch.matching-strategy=ant_path_matcher'.`

  > spring.mvc.pathmatch.matching-strategy=ant_path_matcher

## 内容协商

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/12/14/85200_image-1702559859264-1.png" alt="image" style="zoom: 80%;" />

  1.1. 基于请求头内容协商：**（默认开启）**
    1.1.1. 客户端向服务端发送请求，携带HTTP标准的Accept请求头。
      1.1.1.1. Accept: application/json、text/xml、text/yaml
      1.1.1.2. 服务端根据客户端请求头期望的数据类型进行动态返回
  1.2. 基于请求参数内容协商：**（需要开启）**
    1.2.1. 发送请求 GET /projects/spring-boot?`format=json` 
    1.2.2. 匹配到 @GetMapping("/projects/spring-boot") 
    1.2.3. 根据参数协商，优先返回 json 类型数据【需要开启参数匹配设置】
    1.2.4. 发送请求 GET /projects/spring-boot?format=xml,优先返回 xml 类型数据

1. 引入支持写出xml内容依赖

   ~~~properties
   <dependency>
       <groupId>com.fasterxml.jackson.dataformat</groupId>
       <artifactId>jackson-dataformat-xml</artifactId>
   </dependency>
   ~~~

2. 标注注解

~~~java
@JacksonXmlRootElement  // 可以写出为xml文档
@Data
public class Person {
    private Long id;
    private String userName;
    private String email;
    private Integer age;
}

~~~

3. 开启基于请求参数的内容协商

~~~properties
#开启基于请求参数的内容协商功能。 默认参数名：format
spring.mvc.contentnegotiation.favor-parameter=true

# 指定内容协商时使用的参数名。默认是 format
spring.mvc.contentnegotiation.parameter-name=type
~~~

![image-20231214214334717](https://gitee.com/hollis7/pictures/raw/master/2023/12/14/16124_image-20231214214334717.png)

### 自定义内容返回

`@ResponseBody`由`HttpMessageConverter`处理

1. 如果controller方法的返回值标注了 `@ResponseBody `注解

1. 1. 请求进来先来到`DispatcherServlet`的`doDispatch()`进行处理
   2. 找到一个 `HandlerAdapter `适配器。利用适配器执行目标方法
   3. `RequestMappingHandlerAdapter`来执行，调用`invokeHandlerMethod（）`来执行目标方法
   4. 目标方法执行之前，准备好两个东西

1. 1. 1. `HandlerMethodArgumentResolver`：参数解析器，确定目标方法每个参数值
      2. `HandlerMethodReturnValueHandler`：返回值处理器，确定目标方法的返回值改怎么处理

1. 1. `RequestMappingHandlerAdapter` 里面的`invokeAndHandle()`真正执行目标方法
   2. 目标方法执行完成，会返回**返回值对象**
   3. **找到一个合适的返回值处理器** `HandlerMethodReturnValueHandler`
   4. 最终找到 `RequestResponseBodyMethodProcessor`能处理 标注了 `@ResponseBody`注解的方法
   5. `RequestResponseBodyMethodProcessor` 调用`writeWithMessageConverters `,利用`MessageConverter`把返回值写出去

### 使用yaml返回

导入依赖

~~~xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>
~~~

把对象写出成YAML（**展示，可忽略**）

~~~java
public static void main(String[] args) throws JsonProcessingException {
        Person person = new Person();
        person.setId(1L);
        person.setUserName("张三");
        person.setEmail("aaa@qq.com");
        person.setAge(18);
//        不写yaml文件开头
        YAMLFactory factory = new YAMLFactory().disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER);
        ObjectMapper mapper = new ObjectMapper(factory);
        String s = mapper.writeValueAsString(person);
        System.out.println(s);
    }
~~~

编写配置，新增一种内容形式

~~~properties
#新增一种媒体类型
spring.mvc.contentnegotiation.media-types.yaml=text/yaml
~~~

增加`HttpMessageConverter`组件，专门负责把对象写出为yaml格式

~~~java
@Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                //自己写新的规则。
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/a/", "classpath:/b/")
                        .setCacheControl(CacheControl.maxAge(1180, TimeUnit.SECONDS));

            }

            @Override//配置一个能把对象转为yaml的messageConverter
            public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
                //这里是springboot默认配置的，如果不加@Bean注解，会报错。
                converters.add(new MyYamlHttpMessageConverter());
            }
        };
    }
~~~

~~~java
public class MyYamlHttpMessageConverter extends AbstractHttpMessageConverter<Object> {

    private ObjectMapper mapper = null;//把对象转成yaml

    public MyYamlHttpMessageConverter(){
        //告诉SpringBoot这个MessageConverter支持哪种媒体类型  //媒体类型
        super(new MediaType("text", "yaml", StandardCharsets.UTF_8));
        YAMLFactory factory = new YAMLFactory().disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER);
        this.mapper = new ObjectMapper(factory);
    }
    @Override
    protected boolean supports(Class<?> clazz) {
        //只要是对象类型，不是基本类型
        return true;
    }

    @Override
    protected Object readInternal(Class<?> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    @Override//@ResponseBody 把对象怎么写出去
    protected void writeInternal(Object methodReturnValue, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        //try-with写法，自动关流
        try (OutputStream body = outputMessage.getBody()) {
            this.mapper.writeValue(body, methodReturnValue);
        }

    }
}
~~~

#### 总结

- 配置媒体类型支持: 

- - `spring.mvc.contentnegotiation.media-types.yaml=text/yaml`

- 编写对应的`HttpMessageConverter`，要告诉Boot这个支持的媒体类型

- - 按照`MyYamlHttpMessageConverter`的示例

- 把MessageConverter组件加入到底层

- - 容器中放一个``WebMvcConfigurer`` 组件，并配置底层的`MessageConverter`

## 模板引擎

![image.png](https://cdn.nlark.com/yuque/0/2023/png/1613913/1681354523290-b89d7e0d-b9aa-40f5-8d22-d3d09d02b136.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_28%2Ctext_5bCa56GF6LC3IGF0Z3VpZ3UuY29t%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

### thymeleaf初试

**controller**

~~~java
@Controller
public class WelcomeController {

    @GetMapping("well")
    public String hello(@RequestParam("name")String name, Model model){
        model.addAttribute("msg",name);
        return "welcome";
    }
}
~~~

**welcome.html**

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>你好：<span th:text="${msg}"></span></h1>
</body>
</html>
~~~

> span 标签的作用是用于显示文本内容，并可以通过 th:text 指令将变量的值替换为文本内容。

### thymeleaf基础语法

1. `th:xxx`：动态渲染指定的 html 标签属性值、或者th指令（遍历、判断等）

● th:text：将一切内容都识别为纯文本，不会对 HTML 标签进行解析。
  ○ th:utext：会对 HTML 标签进行解析，并将其生效。。
● th:属性：标签指定属性渲染
● th:attr：标签任意属性渲染
● th:ifth:each...：其他th指令

~~~html
<h1 th:text="${msg}">哈哈</h1>
<h1 th:utext="${msg}">呵呵</h1>
~~~

![image-20231218105441046](https://gitee.com/hollis7/pictures/raw/master/2023/12/18/27187_image-20231218105441046.png)

~~~html
th:任意html属性
<img th:src="${imgUrl}" src="4.jpg" style="width: 300px;"/>
~~~

~~~python
model.addAttribute("imgUrl","static/3.jpg");
model.addAttribute("style","width: 400px");
~~~

~~~html
th：其他指令
<img th:src="${imgUrl}" th:style="${style}" th:if="${show}">
~~~

~~~java
 model.addAttribute("show",true);
~~~

### 自动添加根路径

~~~properties
# 项目的根路径
server.servlet.context-path=/demo
~~~

在浏览器输入url：`/demo/well`后，自动在路径下加载`/demo/static/3.jpg`
~~~html
2.jpg  @{} 专门用来取各种路径
<img src="/static/3.jpg" style="width:300px;" th:src="@{/static/3.jpg}">
~~~

### 遍历

**controller**

~~~java
@GetMapping("/list")
    public String list(Model model) {
        List<Person> list = Arrays.asList(
                new Person(1L, "张三1", "", 15, "pm"),
                new Person(3L, "张三2", "zs2@qq.com", 16, "pm"),
                new Person(4L, "张三333", "zs3@qq.com", 17, "pm"),
                new Person(7L, "张三444", "zs4@qq.com", 18, "admin"),
                new Person(8L, "张三5", "zs5@qq.com", 19, "hr")
        );
        model.addAttribute("persons",list);

//        int i = 10/0;
        return "list";
    }
~~~

**html**

~~~html
<table class="table">
    <thead>
    <tr>
        <th scope="col">#</th>
        <th scope="col">名字</th>
        <th scope="col">邮箱</th>
        <th scope="col">年龄</th>
        <th scope="col">角色</th>
        <th scope="col">状态信息</th>
    </tr>
    </thead>
    <tbody>
    <tr th:each="person,stats:${persons}">
        <th scope="row" th:text="${person.id}"></th>
        <td th:text="${person.userName}"></td>
        <td>[[${person.email}]]</td>
        <td>[[${person.age}]]</td>
        <td th:text="${person.role}"></td>
        <td>
            index:[[${stats.index}]]<br/>
            count:[[${stats.count}]]<br/>
            size(总数量):[[${stats.size}]]<br/>
            current(当前对象): [[${stats.current}]] <br/>
            even(true)/odd(false): [[${stats.even}]] <br/>
            first: [[${stats.first}]] <br/>
            last: [[${stats.last}]] <br/>
        </td>
    </tr>

    </tbody>
</table>
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/12/20/32173_image-20231220110146251.png" alt="image-20231220110146251" style="zoom: 67%;" />

> 可以在bootstrap上复制渲染的css，js链接，点击docss查找

**iterStat 有以下属性：**

- index：当前遍历元素的索引，从0开始
- count：当前遍历元素的索引，从1开始
- size：需要遍历元素的总数量
- current：当前正在遍历的元素对象
- even/odd：是否偶数/奇数行
- first：是否第一个元素
- last：是否最后一个元素

#### th:switch

~~~html
 <td th:switch="${person.role}">
            <button th:case="'admin'" type="button" class="btn btn-danger">管理员</button>
            <button th:case="'pm'" type="button" class="btn btn-primary">项目经理</button>
            <button th:case="'hr'" type="button" class="btn btn-default">人事</button>
</td>
~~~

#### th:if

~~~html
<td th:if="${#strings.isEmpty(person.email)}" th:text="'联系不上'">  </td>
<td th:if="${not #strings.isEmpty(person.email)}" th:text="${person.email}">  </td>
~~~

#### 属性优先级

- 片段
- 遍历
- 判断

#### 对象选择

用`*`选择遍历的对象

~~~html
<tbody>
    <tr th:each="person,stats:${persons}" th:object="${person}">
        <th scope="row" th:text="${person.id}"></th>
<!--        <td th:text="${person.userName}"></td>-->
        <td th:text="*{userName}"></td>
~~~

需要在遍历那里加入 `th:object="${person}`，然后用`*`代替：`th:text="*{userName}"`

### 模板布局

- 定义模板： `th:fragment`

**commom.html**

~~~html
<!--抽取的判断，名字叫 myheader-->
<header th:fragment="myheader" class="p-3 text-bg-dark">
...
</header>
~~~

- 引用模板：`~{templatename::selector}`

**index.html**

~~~html
<!--导航  使用公共部分进行替换-->
<!--  ~{ 模板名 :: 片段名} -->
<div th:replace="~{common::myheader}"></div>
~~~

## 热启动

~~~java
<!--热启动功能 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
~~~

## 国际化

Spring Boot 在类路径根下查找`messages`资源绑定文件。文件名为：`messages.properties`

多语言可以定义多个消息文件，命名为messages_区域代码.properties。如：

- messages.properties：默认
- messages_zh_CN.properties：中文环境
- messages_en_US.properties：英语环境

在程序中可以自动注入 MessageSource组件，获取国际化的配置项值

在页面中可以使用表达式  `#{}`获取国际化的配置项值

![image-20231225111916158](https://gitee.com/hollis7/pictures/raw/master/2023/12/25/99634_image-20231225111916158.png)

**message.properties**

![image-20231225111952638](https://gitee.com/hollis7/pictures/raw/master/2023/12/25/54508_image-20231225111952638.png)

**messages_zh_CN.properties**

![image-20231225112104173](https://gitee.com/hollis7/pictures/raw/master/2023/12/25/39480_image-20231225112104173.png)

**common.html**

~~~html
<div class="text-end">
<!--国际化-->
<button type="button" class="btn btn-outline-light me-2" th:text="#{login}"></button>
<button type="button" class="btn btn-warning" th:text="#{sign}"></button>
</div>
~~~

## 错误处理

<img src="https://cdn.nlark.com/yuque/0/2023/svg/1613913/1681723795095-828d2034-1e6c-4d98-8e47-573dd6b5463b.svg" alt="未命名绘图.svg" style="zoom:67%;" />

### 默认机制

**错误处理的自动配置**都在`ErrorMvcAutoConfiguration`中，两大核心机制：

- 1. SpringBoot 会**自适应****处理错误**，**响应页面**或**JSON数据**
- 2. **SpringMVC的错误处理机制**依然保留，**MVC处理不了**，才会**交给boot进行处理**

  

- 发生错误以后，转发给/error路径，SpringBoot在底层写好一个 BasicErrorController的组件，专门处理这个请求

- 错误页面是这么解析到的

~~~java
//1、解析错误的自定义视图地址
ModelAndView modelAndView = resolveErrorView(request, response, status, model);
//2、如果解析不到错误页面的地址，默认的错误页就是 error
return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
~~~

### 规则

1. **解析一个错误页**

1. a. 如果发生了500、404、503、403 这些错误

1. 1. 1. 如果有**模板引擎**，默认在 `classpath:/templates/error/**精确码.html**`
      2. 如果没有模板引擎，在静态资源文件夹下找  `**精确码.html**`

1. b. 如果匹配不到`精确码.html`这些精确的错误页，就去找`5xx.html`，`4xx.html`**模糊匹配**

1. 1. 1. 如果有模板引擎，默认在 `classpath:/templates/error/5xx.html`
      2. 如果没有模板引擎，在静态资源文件夹下找  `5xx.html`

2. 如果模板引擎路径`templates`下有 `error.html`页面，就直接渲染



## 自动配置原理

1. `ServletWebServerFactoryAutoConfiguration` 自动配置了嵌入式容器场景
2. 绑定了`ServerProperties`配置类，所有和服务器有关的配置 `server`
3. `ServletWebServerFactoryAutoConfiguration` 导入了 嵌入式的三大服务器 `Tomcat`、`Jetty`、`Undertow`

1. 1. 导入 `Tomcat`、`Jetty`、`Undertow` 都有条件注解。系统中有这个类才行（也就是导了包）
   2. 默认  `Tomcat`配置生效。给容器中放 TomcatServletWebServerFactory
   3. 都给容器中 `ServletWebServerFactory`放了一个 **web服务器工厂（造web服务器的）**
   4. **web服务器工厂 都有一个功能，**`getWebServer`获取web服务器
   5. TomcatServletWebServerFactory 创建了 tomcat。

1. ServletWebServerFactory 什么时候会创建 webServer出来。
2. `ServletWebServerApplicationContext`ioc容器，启动的时候会调用创建web服务器
3. Spring**容器刷新（启动）**的时候，会预留一个时机，刷新子容器。`onRefresh()`
4. refresh() 容器刷新 十二大步的刷新子容器会调用 `onRefresh()`；



> Web场景的Spring容器启动，在onRefresh的时候，会调用创建web服务器的方法。
>
> Web服务器的创建是通过WebServerFactory搞定的。容器中又会根据导了什么包条件注解，启动相关的 服务器配置，默认`EmbeddedTomcat`会给容器中放一个 `TomcatServletWebServerFactory`，导致项目启动，自动创建出Tomcat。

