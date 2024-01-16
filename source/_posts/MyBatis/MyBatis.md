---
title: MyBatis
categories:
- mybatis
tags:
- java 
- mybatis
---
<meta name="referrer" content="no-referrer"/>

## MyBaits中文网

[MyBatis](https://mybatis.net.cn/)

## MyBatis快速入门

* 数据库表（tb_user）及数据准备

  ```sql
  create database mybatis;
  use mybatis;
  
  drop table if exists tb_user;
  
  create table tb_user(
  	id int primary key auto_increment,
  	username varchar(20),
  	password varchar(20),
  	gender char(1),
  	addr varchar(30)
  );
  
  
  
  INSERT INTO tb_user VALUES (1, 'zhangsan', '123', '男', '北京');
  INSERT INTO tb_user VALUES (2, '李四', '234', '女', '天津');
  INSERT INTO tb_user VALUES (3, '王五', '11', '男', '西安');
  ```

* 导入依赖，pom.xml，粘贴logback.xml到resource文件夹下

* 编写mybatis-config.xml文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--数据库连接信息-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="hdb2205"/>
            </dataSource>
        </environment>
    </environments>
    <!--加载sql映射文件-->
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
~~~

* 编写UserMapper.xml文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="test">
    <select id="selectAll" resultType="com.ithdb.pojo.User">
        select * from tb_user;
    </select>
</mapper>
~~~

- 主程序

~~~java
public class MyBatisDemo {
    public static void main(String[] args) throws IOException {
        //1. 加载mybatis的核心配置文件，获取 SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象，用它来执行sql
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //3. 执行sql
        List<User> users = sqlSession.selectList("test.selectAll");

        System.out.println(users);

        //4. 释放资源
        sqlSession.close();
    }
}
~~~

## 连接数据库

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/12/71335_image-20240112095356363.png" alt="image-20240112095356363" style="zoom:50%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/12/49567_image-20240112095507785.png" alt="image-20240112095507785" style="zoom: 50%;" />

**刷新一下可以看到数据库的表**

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/12/47675_image-20240112095851919.png" alt="image-20240112095851919" style="zoom: 67%;" />

**idea中使用sql语句进行查询**

![image-20240112100335351](https://gitee.com/hollis7/pictures/raw/master/2024/01/12/57896_image-20240112100335351.png)

## 使用Mapper代理方式完成入门案例

1. 定义与SQL映射文件同名的Mapperf接口，并且将Mapper接口和SQL映射文件放置在同一目录下

![image-20240112102522431](https://gitee.com/hollis7/pictures/raw/master/2024/01/12/70576_image-20240112102522431.png)

这里一定要用`com/inhdb/mapper`创建包，而不是`com.inhdb.mapper`，否则编译后没有文件夹层次结构，只是一个`com.inhdb.mapper`文件

2. 设置SQL映射文件的namespace属性为Mapper接口全限定名

~~~java
<mapper namespace="com.ithdb.mapper.UserMapper">
    <select id="selectAll" resultType="com.ithdb.pojo.User">
        select *
        from tb_user;
    </select>
</mapper>
~~~

3. 在Mapper接口中定义方法，方法名就是SQL映射文件中sql语句的id,并保持参数类型和返回值类型一致

~~~java
public interface UserMapper {
    List<User> selectAll();
}
~~~

4. 修改mybatis-config.xml的UserMapper.xml地址

~~~xml
<!--加载sql映射文件-->
<mappers>
    <mapper resource="com/ithdb/mapper/UserMapper.xml"/>
</mappers>
~~~

5. 编码

~~~java
 //3.1 获取UserMapper接口的代理对象
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
List<User> users = userMapper.selectAll();
~~~

### 小技巧

只要遵循了`Mapper接口和SQL映射文件放置在同一目录下`（编译过后在同一目录下），就可以用包扫描的方法进行配置

`mybatis-config.xml`

~~~java
 <mappers>
     <!--Mapper代理方式，识别到sql映射文件-->
    <package name="com.ithdb.mapper"/>
 </mappers>
~~~

## MyBatis核心配置文件

### environment

environments：配置数据库连接环境信息。可以配置多个environment，通过default属性切换不同的environment

### typeAliases

指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean

```xml
<typeAliases>
    <package name="com.ithdb.pojo"/>
</typeAliases>
```

为com.ithdb.pojo目录下的实体取别名，默认小写如：user

`UserMapper.xml`

~~~java
 <select id="selectAll" resultType="user">
        select *
        from tb_user;
 </select>
~~~

## 增删改查环境准备

准备tb_brand(mybatis数据库下)

~~~sql
-- 删除tb_brand表
drop table if exists tb_brand;
-- 创建tb_brand表
create table tb_brand
(
    -- id 主键
    id           int primary key auto_increment,
    -- 品牌名称
    brand_name   varchar(20),
    -- 企业名称
    company_name varchar(20),
    -- 排序字段
    ordered      int,
    -- 描述信息
    description  varchar(100),
    -- 状态：0：禁用  1：启用
    status       int
);
-- 添加数据
insert into tb_brand (brand_name, company_name, ordered, description, status)
values ('三只松鼠', '三只松鼠股份有限公司', 5, '好吃不上火', 0),
       ('华为', '华为技术有限公司', 100, '华为致力于把数字世界带入每个人、每个家庭、每个组织，构建万物互联的智能世界', 1),
       ('小米', '小米科技有限公司', 50, 'are you ok', 1);


SELECT * FROM tb_brand;
~~~

### 大小驼峰解决方案

springboot有更好的方案

~~~xml
<resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"/>
        <result column="company_name" property="companyName"/>
</resultMap>
<select id="selectAll" resultMap="brandResultMap">
    select *
    from tb_brand;
</select>
~~~

## 查看详情

~~~xml
 <!--
        * 参数占位符：
            1. #{}:会将其替换为 ?，为了防止SQL注入
            2. ${}：拼sql。会存在SQL注入问题
            3. 使用时机：
                * 参数传递的时候：#{}
                * 表名或者列名不固定的情况下：${} 会存在SQL注入问题

         * 参数类型：parameterType：可以省略
         * 特殊字符处理：
            1. 转义字符：
            2. CDATA区:
    -->
<select id="selectById" resultMap="brandResultMap">
        select *
        from tb_brand
        where id = #{id};
</select>
~~~

### 特殊字符转义

法一，转义：

~~~xml
<select id="selectById" resultMap="brandResultMap">
        select *
        from tb_brand
        where id &lt; #{id};
</select>
~~~

如<对应`&lt;`

法二，CDATA

~~~xml
<select id="selectById" resultMap="brandResultMap">
        select *
        from tb_brand
        where id
            <![CDATA[
            <
            ]]>
            #{id};
    </select>
~~~

## 条件查询

~~~xml
 /**
     * 条件查询
     * * 参数接收
     * 1. 散装参数：如果方法中有多个参数，需要使用@Param("SQL参数占位符名称")
     * 2. 对象参数:对象的属性名称要和参数占位符名称一致
     * 3. map集合参数
     */
~~~

~~~java
int status = 1;
String companyName = "华为";
String brandName = "华为";
// 处理参数
companyName = "%" + companyName + "%";
brandName = "%" + brandName + "%";

//封装对象
//        Brand brand = new Brand();
//        brand.setStatus(status);
//        brand.setCompanyName(companyName);
//        brand.setBrandName(brandName);

Map map = new HashMap();
map.put("status", status);
map.put("companyName", companyName);
map.put("brandName", brandName);
~~~

~~~java
//    List<Brand> selectByCondition(@Param("status") int status, @Param("companyName") String companyName, @Param("brandName") String brandName);
//    List<Brand> selectByCondition(Brand brand);
    List<Brand> selectByCondition(Map map);
~~~

## 动态多条件查询

~~~xml
<!--
       动态条件查询
           * if: 条件判断
               * test：逻辑表达式
           * 问题：
               * 恒等式
               * <where> 替换 where 关键字
-->
~~~

~~~xml
<select id="selectByCondition" resultMap="brandResultMap">
    select *
    from tb_brand
    where
    <if test="status!=null">
        status = #{status}
    </if>
    <if test="companyName!=null and companyName!='' ">
        and company_name like #{companyName}
    </if>
    <if test="brandName!=null and brandName!='' ">
        and brand_name like #{brandName}
    </if>
</select>
~~~

如果第一个status后面参数不为空就会存在sql语法错误

~~~xml
 <select id="selectByCondition" resultMap="brandResultMap">
        select *
        from tb_brand
        /* where 1=1 */
        <where>
            <if test="status!=null">
                and status = #{status}
            </if>
            <if test="companyName!=null and companyName!='' ">
                and company_name like #{companyName}
            </if>
            <if test="brandName!=null and brandName!='' ">
                and brand_name like #{brandName}
            </if>
        </where>
    </select>
~~~

> 存在无参数全查询问题

## 单条件动态查询

~~~java
<select id="selectByConditionSingle" resultMap="brandResultMap">
    select *
    from tb_brand
    where
    <choose><!--相当于switch-->
        <when test="status!=null"><!--相当于case-->
            status = #{status}
        </when>
        <when test="companyName!=null and companyName!=''">
            company_name like #{companyName}
        </when>
        <when test="brandName != null and brandName != ''"><!--相当于case-->
            brand_name like #{brandName}
        </when>
        <otherwise>
            1=1
        </otherwise>
    </choose>
</select>
~~~

otherwise可以通过`<where>`包裹进行替换

## 添加或者修改

### 添加

BrandMapper.xml

~~~xml
 <insert id="add">
        insert into tb_brand (brand_name, company_name, ordered, description, status)
        values (#{brandName}, #{companyName}, #{ordered}, #{description}, #{status});
</insert>
~~~

**注意需要提交或者开始处开启自动提交**

~~~java
 //3. 获取Mapper接口的代理对象
BrandMapper brandMapper = sqlSession.getMapper(BrandMapper.class);

//4. 执行方法
brandMapper.add(brand);
//提交事务,否则插入的数据会回滚
sqlSession.commit();
~~~

~~~java
 //2. 获取SqlSession对象
SqlSession sqlSession = sqlSessionFactory.openSession(true);
~~~

### 添加项获取主键如id值

使用useGeneratedKeys、keyProperty绑定插入数据的主键到id

~~~xml
<insert id="add" useGeneratedKeys="true" keyProperty="id">
    insert into tb_brand (brand_name, company_name, ordered, description, status)
    values (#{brandName}, #{companyName}, #{ordered}, #{description}, #{status});
</insert>
~~~

~~~java
//4. 执行方法
brandMapper.add(brand);
Integer id = brand.getId();
System.out.println(id);
~~~

## 修改-动态

使用set标签避免sql参数不全，多“，”造成语法错误

~~~xml
<update id="update">
    update tb_brand
    <set>
        <if test="brandName != null and brandName != ''">
            brand_name = #{brandName},
        </if>
        <if test="companyName != null and companyName != ''">
            company_name = #{companyName},
        </if>
        <if test="ordered != null">
            ordered = #{ordered},
        </if>
        <if test="description != null and description != ''">
            description = #{description},
        </if>
        <if test="status != null">
            status = #{status}
        </if>
    </set>

    where id=#{id};
</update>
~~~

## 删除

### 单个

~~~xml
<delete id="deleteById">
    delete
    from tb_brand
    where id = #{id};
</delete>
~~~

### 批量删除

sql原语

~~~sql
delete from tb_brand where id in (?,?,?);
~~~

mybatis改写

```java
void deleteByIds(@Param("ids") int[] ids);
```

~~~xml
 <!--
        mybatis会将数组参数，封装为一个Map集合。
            * 默认：array = 数组
            * 使用@Param注解改变map集合的默认key的名称
-->
<delete id="deleteByIds">
    delete
    from tb_brand
    where id in (
    <foreach collection="ids" item="id" separator=",">
        #{id}
    </foreach>
    )
</delete>
~~~

separator的目的是为了分隔多个id

还可以将where id in (?,?,?)中的括号用这种形式改变

~~~xml
<foreach collection="ids" item="id" separator="," open="(" close=")">
    #{id}
</foreach>
~~~

## 参数传递

**多个参数：封装为Map集合,可以使用@Param注解，替换Map集合中默认的arg键名**

~~~java
 User select(@Param("username") String username, @Param("password") String password);
~~~

## 注解开发

简单的sql语句用注解，复杂的用xml配置方法

~~~java
@Select("select * from tb_user where id = #{id};")
User selectById(int id);
~~~

## springboot3使用MyBatis

### 步骤

1、用spring initial创建module

2、选中初始依赖

![image-20240114212007854](https://gitee.com/hollis7/pictures/raw/master/2024/01/14/61655_image-20240114212007854.png)

`MyBatis Framework`和`MySQL Driver`在SQL的依赖中

3、`application.properties`配置数据源

~~~properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=hdb2205
~~~

4、创建pojo类

~~~java
@Data
public class TUser {
    private Long id;
    private String loginName;
    private String nickName;
    private String passwd;
}
~~~

5、创建一个接口`UserMapper`，resource文件夹下创建`mapper`文件夹，利用MybatisX这个插件生产`UserMapper.xml`文件（选中接口点击 more actions，选中生产xml，选中xml文件生成的位置）

![image-20240114213800163](https://gitee.com/hollis7/pictures/raw/master/2024/01/14/28490_image-20240114213800163.png)

![image-20240114213851958](https://gitee.com/hollis7/pictures/raw/master/2024/01/14/43338_image-20240114213851958.png)

![image-20240114214002500](https://gitee.com/hollis7/pictures/raw/master/2024/01/14/98218_image-20240114214002500.png)

6、在接口中编辑方法，利用插件自动生成statement

~~~java
 TUser getUserById(@Param("id") Long id);
~~~

~~~xml
 <select id="getUserById" resultType="com.hdb.boot3mybatis.bean.TUser">
        select *
        from t_user
        where id = #{id}
</select>
~~~

7、主程序中加入`MapperScan`，扫描接口包的位置

~~~java
@MapperScan(basePackages = "com/hdb/boot3mybatis/mapper")
~~~

8、配置xxxmapper.xml的位置

~~~properties
mybatis.mapper-locations=classpath:/mapper/*.xml
~~~

9、配置驼峰命名

~~~properties
mybatis.configuration.map-underscore-to-camel-case=true
~~~

