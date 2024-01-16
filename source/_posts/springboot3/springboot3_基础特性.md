---
title: springboot3 基础特性
categories:
  - springboot3
tags:
  - springboot3
---
<meta name="referrer" content="no-referrer"/>

## 自定义 banner

1. 类路径添加banner.txt或设置spring.banner.location就可以定制 banner
2. 推荐网站：[Spring Boot banner 在线生成工具，制作下载英文 banner.txt，修改替换 banner.txt 文字实现自定义，个性化启动 banner-bootschool.net](https://www.bootschool.net/ascii)

<!--more-->

## 环境隔离

1、标识环境

```
1）、区分出几个环境： dev（开发环境）、test（测试环境）、prod（生产环境）
2）、指定每个组件在哪个环境下生效； default环境：默认环境
	通过： @Profile({"test"})标注
	组件没有标注@Profile代表任意时候都生效
3）、默认只有激活指定的环境，这些组件才会生效。
```

2、激活环境

```
 配置文件激活：spring.profiles.active=dev；
 命令行激活： java -jar xxx.jar  --spring.profiles.active=dev
 集成环境如下图
```

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/04/72330_image-20240104164250109.png" alt="image-20240104164250109" style="zoom: 50%;" />

~~~bash
--spring.profiles.active=dev
~~~

3、环境包含

```properties
#包含指定环境，不管你激活哪个环境，这个都要有。总是要生效的环境
spring.profiles.include=prod
```

### 最佳实战

- **生效的环境** = **激活的环境/默认环境**  + **包含的环境**
- 项目里面这么用

- - 基础的配置`mybatis`、`log`、`xxx`：写到**包含环境中**
  - 需要动态切换变化的 `db`、`redis`：写到**激活的环境中**

### Profile 分组

~~~bash
spring.profiles.group.haha=db,log
spring.profiles.group.hehe=mq
~~~

## profile配置文件

3、配置文件怎么使用Profile功能

1）、application.properties： 主配置文件。任何情况下都生效

2）、其他Profile环境下命名规范：  application-{profile标识}.properties：

比如：application-dev.properties

3）、激活指定环境即可：  配置文件激活、命令行激活

4）、效果：

**项目的所有生效配置项 = 激活环境配置文件的所有项 + 主配置文件和激活文件不冲突的所有项**

**如果发生了配置冲突，以激活的环境配置文件为准。**

application-{profile标识}.properties 优先级高于 application.properties

主配置和激活的配置都生效，优先以激活的配置为准

## 配置优先级

命令行`> `配置文件`> `springapplication配置



**配置文件优先级**如下：(**后面覆盖前面**)

1. **jar 包内**的application.properties/yml
2. **jar 包内**的application-{profile}.properties/yml
3. **jar 包外**的application.properties/yml
4. **jar 包外**的application-{profile}.properties/yml



最终效果：优先级由高到低，前面覆盖后面

命令行 > 包外config直接子目录 > 包外config目录 > 包外根目录 > 包内目录

![未命名绘图](C:\data\mysoftware\Typora\typoraPicture\未命名绘图.svg)

![未命名绘图.svg](https://cdn.nlark.com/yuque/0/2023/svg/1613913/1682073869709-2cba18c8-55bd-4bf1-a9df-ac784e30d89a.svg)

## 属性占位符

配置文件中可以使用 `${name:default}`形式取出之前配置过的值。

~~~properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application written by ${username:Unknown}
~~~

~~~java
@Value("${app.name:中国mooc}")
String appName;
~~~



## 单元测试

@SpringBootTest