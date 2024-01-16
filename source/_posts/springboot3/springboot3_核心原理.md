---
title: springboot3 核心原理
categories:
  - springboot3
tags:
  - springboot3
---
<meta name="referrer" content="no-referrer"/>

## springboot3笔记资料地址

[springboot3-notes](https://www.yuque.com/leifengyang/springboot3)

<!--more-->

## 事件和监听器

1. 自定义`SpringApplicationRunListener`来**监听事件**；

1. 1. 编写`SpringApplicationRunListener` **实现类**
   2. 在 `META-INF/spring.factories` 中配置 `org.springframework.boot.SpringApplicationRunListener=自己的Listener`，还可以指定一个**有参构造器**，接受两个参数`(SpringApplication application, String[] args)`

2. ```properties
   # spring.factories
   org.springframework.boot.SpringApplicationRunListener=com.hdb.core.listener.MyAppListener
   ```

3. 3. springboot 在`spring-boot.jar`中配置了默认的 事件监听器

~~~java
/**
 * Listener 先要从 META-INF/spring.factories 读到
 *
 * 1、引导： 利用 BootstrapContext 引导整个项目启动
 *      starting：              应用开始，SpringApplication的run方法一调用，只要有了 BootstrapContext 就执行
 *      environmentPrepared：   环境准备好（把启动参数等绑定到环境变量中），但是ioc还没有创建；【调一次】
 * 2、启动：
 *      contextPrepared：       ioc容器创建并准备好，但是sources（主配置类）没加载。并关闭引导上下文；组件都没创建  【调一次】
 *      contextLoaded：         ioc容器加载。主配置类加载进去了。但是ioc容器还没刷新（我们的bean没创建）。
 *      =======截止以前，ioc容器里面还没造bean呢=======
 *      started：               ioc容器刷新了（所有bean造好了），但是 runner 没调用。
 *      ready:                  ioc容器刷新了（所有bean造好了），所有runner调用完了。
 * 3、运行
 *     以前步骤都正确执行，代表容器running。
 */
~~~

### 生命周期全流程

![image](https://gitee.com/hollis7/pictures/raw/master/2024/01/04/28084_image-1704377032144-1.png)

## 事件触发时机

- **ApplicationListener：    感知全阶段：基于事件机制，感知事件。 一旦到了哪个阶段可以做别的事**

- - `@Bean`或`@EventListener`： `事件驱动`
  - `SpringApplication.addListeners(…)`或 `SpringApplicationBuilder.listeners(…)`
  - `META-INF/spring.factories`

- ![image](https://gitee.com/hollis7/pictures/raw/master/2024/01/05/68286_image-1704437922998-1.png)

~~~java
public class MyListener implements ApplicationListener<ApplicationEvent> {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("=====事件====到达===="+event);
    }
}
~~~

~~~properties
org.springframework.context.ApplicationListener=com.hdb.core.listener.MyListener
~~~

应用事件发送顺序如下：

![image 2](https://gitee.com/hollis7/pictures/raw/master/2024/01/05/52391_image 2.png)

## SpringBoot 事件驱动开发

![image3](https://gitee.com/hollis7/pictures/raw/master/2024/01/05/31052_image3.png)

### 事件发布者

~~~java
@Service
public class EventPublisher implements ApplicationEventPublisherAware {
    ApplicationEventPublisher applicationEventPublisher;

    /**
     * 所有事件都可以发
     * @param event
     */
    public void sendEvent(ApplicationEvent event) {
        //调用底层API发送事件
        applicationEventPublisher.publishEvent(event);
    }

    /**
     * 会被自动调用，把真正发事件的底层组组件给我们注入进来
     * @param applicationEventPublisher event publisher to be used by this object
     */
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
~~~

### 登录成功事件

~~~java
/**
 * 登录成功事件。所有事件都推荐继承 ApplicationEvent
 */
public class LoginSuccessEvent extends ApplicationEvent {
    /**
     *  代表是谁登录成了
     * @param source
     */
    public LoginSuccessEvent(UserEntity source) {
        super(source);
    }
}
~~~

### service监听

方法一实现ApplicationListener接口

~~~java
@Service
public class AccountService  implements ApplicationListener<LoginSuccessEvent>{

    public void addAccountScore(String username){
        System.out.println(username +" 加了1分");
    }

    @Override
    public void onApplicationEvent(LoginSuccessEvent event) {
        System.out.println("=====  AccountService  收到事件 =====");
        UserEntity source = (UserEntity) event.getSource();
        addAccountScore(source.getUsername());
    }
}
~~~

方法二，`@EventListener`注解

~~~java
@Service
public class CouponService {

    @EventListener
    public void onEvent(LoginSuccessEvent loginSuccessEvent){
        System.out.println("===== CouponService ====感知到事件"+loginSuccessEvent);
        UserEntity source = (UserEntity) loginSuccessEvent.getSource();
        sendCoupon(source.getUsername());
    }

    public void sendCoupon(String username){
        System.out.println(username + " 随机得到了一张优惠券");
    }
}
~~~

### LoginController

~~~java
@RestController
public class LoginController {

    @Autowired
    AccountService accountService;

    @Autowired
    CouponService couponService;

    @Autowired
    SysService sysService;

    @Autowired
    EventPublisher eventPublisher;

    @GetMapping("/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("passwd")String passwd){
        //业务处理登录
        System.out.println("业务处理登录完成....");
        //TODO 发送事件.
        //1、创建事件信息
        LoginSuccessEvent event = new LoginSuccessEvent(new UserEntity(username, passwd));
        //2、发送事件
        eventPublisher.sendEvent(event);

        return username+"登录成功";
    }
}
~~~

## SPI机制

在Java中，**SPI**的实现方式是通过在`META-INF/services`目录下创建一个以服务接口全限定名为名字的文件，文件中包含实现该服务接口的类的全限定名。当应用程序启动时，Java的SPI机制会自动扫描classpath中的这些文件，并根据文件中指定的类名来加载实现类。

在SpringBoot中，`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

## 功能开关

- 自动配置：全部都配置好，什么都不用管。   自动批量导入

- - 项目一启动，spi文件中指定的所有都加载。

- `@EnableXxxx`：手动控制哪些功能的开启； 手动导入。

- - 开启xxx功能
  - 都是利用 @Import 把此功能要用的组件导入进去

## 理解@SpringBootApplication

`@SpringBootConfiguration`

就是： @Configuration ，容器中的组件，配置类。spring ioc启动就会加载创建这个类对象

`@AutoConfigurationPackage`：扫描主程序包：加载自己的组件

> - 利用 `@Import(AutoConfigurationPackages.Registrar.class)` 想要给容器中导入组件。
> - 把主程序所在的**包**的所有组件导入进来。

`@Import(AutoConfigurationImportSelector.class)`：加载所有自动配置类：加载starter导入的组件

`@ComponentScan`
组件扫描：排除一些组件（哪些不要）
排除前面已经扫描进来的配置类、和自动配置类。

## 完整启动加载流程

![自动配置进阶原理.svg](https://cdn.nlark.com/yuque/0/2023/svg/1613913/1682569555020-b6cbc750-3171-44c6-810f-1c59e590b792.svg)

## 自定义starter

### 1. 配置处理器（省略）

导入配置处理器，配置文件自定义的properties配置都会有提示

~~~properties
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
</dependency>
~~~

> 2023版默认已经有提示了

### 2. 业务代码

~~~java
@ConfigurationProperties(prefix = "robot")  //此属性类和配置文件指定前缀绑定
@Component
@Data
public class RobotProperties {

    private String name;
    private String age;
    private String email;
}
~~~

### 3.1 基本抽取

- 创建starter项目，把公共代码需要的所有依赖导入

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/08/22745_image-20240108195339109.png" alt="image-20240108195339109" style="zoom:67%;" />

- 把公共代码复制进来
- 自己写一个 `RobotAutoConfiguration`，给容器中导入这个场景需要的所有组件

- - 为什么这些组件默认不会扫描进去？
  - **starter所在的包和引入它的项目的主程序所在的包不是父子层级**

- 别人引用这个`starter`，直接导入这个 `RobotAutoConfiguration`,就能把这个场景的组件导入进来

~~~xml
 <!--        自定义starter -->
<dependency>
    <groupId>com.hdb</groupId>
    <artifactId>boot3-08-robot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
~~~
`RobotAutoConfiguration`用`Import`导入了相关的组件，Boot306FeaturesApplication通过`Import`导入了`RobotAutoConfiguration`就可以使用robot相关服务了

~~~java
@Configuration
@Import导入了({RobotController.class, RobotProperties.class, RobotService.class})
public class RobotAutoConfiguration {
}
~~~

~~~java
@Import(RobotAutoConfiguration.class)
public class Boot306FeaturesApplication{...}
~~~

### 3.2 使用@EnableXxx机制

在robot-starter的包里再写一个申明接口，模仿`EnableWebMvc`写法，在Boot306FeaturesApplication上添加注释`@EnableRobot`

~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({RobotAutoConfiguration.class})
public @interface EnableRobot {
}
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/08/32500_image-20240108200942226.png" alt="image-20240108200942226" style="zoom:67%;" />

### 3.3 完全自动配置

- 依赖SpringBoot的SPI机制
- META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports 文件中编写好我们自动配置类的全类名即可

~~~properties
# org.springframework.boot.autoconfigure.AutoConfiguration.imports 
com.hdb.boot3.starter.robot.RobotAutoConfiguration
~~~

