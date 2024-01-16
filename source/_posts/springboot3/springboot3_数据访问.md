---
title: springboot3 数据访问
categories:
  - springboot3
tags:
  - springboot3
---
<meta name="referrer" content="no-referrer"/>

## springboot3笔记资料地址

[springboot3-notes-尚硅谷](https://www.yuque.com/leifengyang/springboot3)

<!--more-->

## mysql

### 启动

```
 mysql -u root -p
```

### 密码

```
hdb2205
```

## 新建module

![image-20240103152054170](https://gitee.com/hollis7/pictures/raw/master/2024/01/03/61239_image-20240103152054170.png)

## 整合SSM场景

1. 创建SSM整合项目
2. 配置数据源

~~~properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=hdb2205
~~~

安装MyBatisX 插件，帮我们生成Mapper接口的xml文件即可

3. `Boot305SsmApplication`加入`@MapperScan`注释

```java
@MapperScan(basePackages = "com/hdb/boot/boot3/ssm/mapper")
```

~~~properties
# 2、配置整合MyBatis
mybatis.mapper-locations=classpath:/mapper/*.xml
# 打开驼峰命名规则，mysql和java中的变量名字命名规则不同
mybatis.configuration.map-underscore-to-camel-case=true
~~~

## 自动配置原理

1. **导入** `mybatis-spring-boot-starter`
2. 配置**数据源**信息
3. 配置mybatis的`**mapper接口扫描**`与`**xml映射文件扫描**`
4. 编写bean，mapper，生成xml，编写sql 进行crud。**事务等操作依然和Spring中用法一样**
5. 效果：

1. 1. 所有sql写在xml中
   2. 所有`mybatis配置`写在`application.properties`下面