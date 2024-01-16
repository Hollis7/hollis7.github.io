---
title: springMVC学习笔记
categories:
  - springMVC
tags:
  - SpirngMVC
  - java
---
<meta name="referrer" content="no-referrer"/>

<!--more-->

## 创建项目

1. 先创建空项目

![image-20230831163022372](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/20184_image-20230831163022372.png)

2. 创建module

等待创建过程

![image-20230831163316649](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/78249_image-20230831163316649.png)

3. 新建java目录

![image-20230831163503073](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/46834_image-20230831163503073.png)

4. 更改per-module bytecode

否则会报：java: 错误: 不支持发行版本 5

![image-20230831200656888](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/95133_image-20230831200656888.png)

> 很显然这种方式很麻烦，每次修改pom.xml文件后，都需要再次去设置。

**法二、在pom.xml文件中配置**

~~~xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>
~~~

## local tomcat配置

提前安装tomcat，`application server`选择安装的目录

![image-20230831202822348](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/11337_image-20230831202822348.png)

![image-20230831144725474](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/84522_image-20230831144725474.png)

![image-20230831144747097](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/83078_image-20230831144747097.png)

## tomcat乱码

### server.xml

~~~java
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
                URIEncoding="UTF-8"
               />
~~~

### logging.properties

~~~java
java.util.logging.ConsoleHandler.encoding =  GBK
~~~

## 总结

浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器，将请求地址和控制器中@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的控制器方法就是处理请求的方法。处理请求的方法需要返回一个字符串类型的视图名称，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应页面

## 新建module后操作

删掉原有deployment，添加新的`springMVC-demo2:war exploded`

![image-20230831200416626](https://gitee.com/hollis7/pictures/raw/master/2023/08/31/36330_image-20230831200416626.png)

## @RequestMapping注解的value属性

@RequestMapping注解的value属性通过请求的请求地址匹配请求映射

@RequestMapping注解的value属性是一个字符串类型的数组，表示该请求映射能够匹配多个请求地址所对应的请求

@RequestMapping注解的value属性必须设置，至少通过请求地址匹配请求映射

## @RequestMapping注解的params属性

@RequestMapping注解的params属性通过请求的请求参数匹配请求映射

@RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系

"param"：要求请求映射所匹配的请求必须携带param请求参数

"!param"：要求请求映射所匹配的请求必须不能携带param请求参数

"param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value

"param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value



## SpringMVC支持ant风格的路径

？：表示任意的单个字符

> java代码中用`/a?a`

*：表示任意的0个或多个字符

\**：表示任意的一层或多层目录

注意：在使用\**时，只能使用/**/xxx的方式

## RequestParam

@RequestParam是将请求参数和控制器方法的形参创建映射关系

@RequestParam注解一共有三个属性：

value：指定为形参赋值的请求参数的参数名

required：设置是否必须传输此请求参数，默认值为true

**若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面报错400**：Required String parameter 'xxx' is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null

## 解决获取请求参数的乱码问题

解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册

~~~xml
<!--配置springMVC的编码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~

> SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效

## 域对象共享数据

ServletAPI, ModelAndView, Model, map, ModelMap

### Model, map, ModelMap关系

~~~java
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
~~~

## jsp上下文路径

更新后的版本，直接使用

~~~xml
<a href="success">success.jsp</a>
~~~

不再使用

~~~xml
<a href="${pageContext.request.contextPath}/success">success.jsp</a>
~~~

### HiddenHttpMethodFilter

由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

a>当前请求的请求方式必须为post

b>当前请求必须传输请求参数_method

满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数_method的值，因此请求参数\_method的值才是最终的请求方式

在web.xml中注册**HiddenHttpMethodFilter** 

~~~xml
<form th:action="@{/user}" method="post">
    <input type="hidden" name="_method" value="PUT">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit" value="修改"><br>
</form>
~~~

~~~xml
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~

> 必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter

## 添加功能流程

在employee_list中点击add功能，通过视图控制器（springMVC.xml)跳转到employee_add界面，输入信息，通过POST方法进入控制器，控制器保存新的员工，重定向到`/employee`重新进入查询所有的员工信息。

## get_post_delete_post

| 操作     | 传统方式         | REST风格                |
| -------- | ---------------- | ----------------------- |
| 查询操作 | getUserById?id=1 | user/1-->get请求方式    |
| 保存操作 | saveUser         | user-->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1-->delete请求方式 |
| 更新操作 | updateUser       | user-->put请求方式      |

## 更新功能流程

在employee_list中点击update功能，跳转到控制器中的getEmployeeById（RequestMethod.GET）查询对应id的employee，将employee信息发送到employee_update网页，网页采用回显，提交信息后跳转到控制器中的updateEmployee（put），更新保存信息，重定向到控制器的getAllEmployee显示所有员工信息。

## @RequestBody

@RequestBody可以获取请求体，需要在控制器方法设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

~~~html
<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit">
</form>
~~~

~~~java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody:"+requestBody);
    return "success";
}
~~~

## RequestEntity

RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

~~~java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
~~~

## @ResponseBody

@ResponseBody用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应到浏览器

> 不再把success解析为html视图

~~~java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
~~~

## SpringMVC处理json

浏览器不能识别java响应的对象，需要转化为json格式

~~~java
@RequestMapping("/testResponseUser")
    @ResponseBody
    public User testResponseUser(){
        return new User(1001, "admin", "123456", 23, "男");
    }
~~~

a>导入jackson的依赖

~~~java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.1</version>
</dependency>
~~~

b>在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的Java对象转换为Json格式的字符串

~~~java
<mvc:annotation-driven />
~~~

c>在处理器方法上使用@ResponseBody注解进行标识

d>将Java对象直接作为控制器方法的返回值返回，就会自动转换为**Json格式的字符串**

## json

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，通常用于在不同系统之间传递和存储数据。JSON的设计目标是简单、易于阅读和编写，同时也易于解析和生成。它以文本形式表示数据，由键值对组成，具有以下特点：

1. 键值对：JSON 数据是由键值对组成的，每个键（也称为属性或字段）都是一个字符串，值可以是字符串、数字、布尔值、数组、对象、null等。键值对之间使用冒号（`:`）分隔，而不同的键值对之间使用逗号（`,`）分隔。
2. 嵌套结构：JSON 支持嵌套结构，可以在值中包含其他键值对，从而构建复杂的数据结构。
3. 数组：JSON 数组是一种有序的值的集合，使用方括号（`[]`）来表示。数组中的元素可以是任何合法的 JSON 数据类型，包括对象、字符串、数字等。
4. 简洁性：JSON 的语法相对简单，易于阅读和编写。这使得它成为了在不同编程语言之间进行数据交换的理想选择。
5. 自我描述性：JSON 数据本身包含数据类型信息，因此接收方可以很容易地理解数据的结构和内容。
6. 广泛应用：JSON 在Web开发中非常常见，用于传输数据、配置文件、API通信等。它也得到了许多编程语言的支持，因此可以轻松地在不同平台上使用。****

~~~json
{
  "name": "John Doe",
  "age": 30,
  "isStudent": false,
  "languages": ["JavaScript", "Python", "C++"],
  "address": {
    "street": "123 Main St",
    "city": "Anytown"
  }
}
~~~

## 重新打包

再webapp下添加了static后，需要重新打包，找到maven中的lifestyle下的package，点击

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/10/30/28028_image-20231030104445931.png" alt="image-20231030104445931" style="zoom: 50%;" /><img src="https://gitee.com/hollis7/pictures/raw/master/2023/10/30/88240_image-20231030104506076.png" alt="image-20231030104506076" style="zoom: 50%;" />

## @RestController注解

`@RestController`注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了`@Controller`注解，并且为其中的每个方法添加了`@ResponseBody`注解

## 文件上传/下载

### 下载

获取ServletContext对象

获取服务器中文件的真实路径

创建输入流

创建字节数组

将流读到字节数组中

创建HttpHeaders对象设置响应头信息

设置要下载方式以及下载文件的名字

设置响应状态码

创建ResponseEntity对象

~~~java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    //获取ServletContext对象
    ServletContext servletContext = session.getServletContext();
    //获取服务器中文件的真实路径
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    //创建输入流
    InputStream is = new FileInputStream(realPath);
    //创建字节数组
    byte[] bytes = new byte[is.available()];
    //将流读到字节数组中
    is.read(bytes);
    //创建HttpHeaders对象设置响应头信息
    MultiValueMap<String, String> headers = new HttpHeaders();
    //设置要下载方式以及下载文件的名字
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    //设置响应状态码
    HttpStatus statusCode = HttpStatus.OK;
    //创建ResponseEntity对象
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    //关闭输入流
    is.close();
    return responseEntity;
}
~~~

## 拦截器

这两条都是对所有请求拦截（springMVC配置）

~~~xml
<bean class="com.hdb.mvc.interceptors.FirstInterceptor"></bean>
<ref bean="firstInterceptor"></ref>
~~~

~~~xml
<mvc:interceptor>
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/>
    <ref bean="firstInterceptor"></ref>
</mvc:interceptor>
<!-- 
	以上配置方式可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求，通过mvc:exclude-mapping设置需要排除的请求，即不需要拦截的请求
-->
~~~

### 拦截器的三个抽象方法

SpringMVC中的拦截器有三个抽象方法：

preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法

postHandle：控制器方法执行之后执行postHandle()

afterComplation：处理完视图和模型数据，渲染视图完毕之后执行afterComplation()

## 基于配置的异常处理

SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver
HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和SimpleMappingExceptionResolver. SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式：

~~~xml
 <!--配置异常处理-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="java.lang.ArithmeticException">error</prop>
            </props>
        </property>
<!--        设置将异常信息共享在请求域中的键-->
        <property name="exceptionAttribute" value="ex"></property>
    </bean>
~~~

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
出现错误
<p th:text="${ex}"></p>
</body>
</html>
~~~

## 注解配置SpringMVC

Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为`AbstractAnnotationConfigDispatcherServletInitializer`，当我们的类扩展了`AbstractAnnotationConfigDispatcherServletInitializer`并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

~~~java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 指定spring的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC的配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则，即url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 添加过滤器
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceRequestEncoding(true);
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
    }
}
~~~

### webconfig

代替SpringMVC的配置文件：

1、扫描组件  2、视图解析器   3、view-controller   4、default-servlet-handler

 5、mvc注解驱动   6、文件上传解析器  7、异常处理    8、拦截器

~~~java
package com.hdb.mvc.config;

import com.hdb.mvc.interceptor.TestInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.*;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;
import org.thymeleaf.spring5.SpringTemplateEngine;
import org.thymeleaf.spring5.view.ThymeleafViewResolver;
import org.thymeleaf.templatemode.TemplateMode;
import org.thymeleaf.templateresolver.ITemplateResolver;
import org.thymeleaf.templateresolver.ServletContextTemplateResolver;

import java.util.List;
import java.util.Properties;


/**
 * 代替SpringMVC的配置文件：
 * 1、扫描组件   2、视图解析器     3、view-controller    4、default-servlet-handler
 * 5、mvc注解驱动    6、文件上传解析器   7、异常处理      8、拦截器
 */
@Configuration
//1、扫描组件
@ComponentScan("com.hdb.mvc.controller")
//5、mvc注解驱动
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    //4、default-servlet-handler
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //8、拦截器
    public void addInterceptors(InterceptorRegistry registry) {
        TestInterceptor testInterceptor = new TestInterceptor();
        registry.addInterceptor(testInterceptor).addPathPatterns("/**");
    }

    //3、view-controller
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
    }

    //6、文件上传解析器
    @Bean
    public MultipartResolver multipartResolver() {
        CommonsMultipartResolver commonsMultipartResolver = new CommonsMultipartResolver();
        return commonsMultipartResolver;
    }

    //7、异常处理

    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties prop =  new Properties();
        prop.setProperty("java.lang.ArithmeticException","error");
        //出现ArithmeticException映射到error。html
        exceptionResolver.setExceptionMappings(prop);
        //exception的值是ArithmeticException中对应的错误信息，共享到请求域中的键
        exceptionResolver.setExceptionAttribute("exception");
        resolvers.add(exceptionResolver);
    }

    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();
        // ServletContextTemplateResolver需要一个ServletContext作为构造参数，可通过WebApplicationContext 的方法获得
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(
                webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    //生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }
}

~~~

## SpringMVC常用组件

- DispatcherServlet：**前端控制器**，不需要工程师开发，由框架提供

作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求

- HandlerMapping：**处理器映射器**，不需要工程师开发，由框架提供

作用：根据请求的url、method等信息查找Handler，即控制器方法

- Handler：**处理器**，需要工程师开发

作用：在DispatcherServlet的控制下Handler对具体的用户请求进行处理

- HandlerAdapter：**处理器适配器**，不需要工程师开发，由框架提供

作用：通过HandlerAdapter对处理器（控制器方法）进行执行

- ViewResolver：**视图解析器**，不需要工程师开发，由框架提供

作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

- View：**视图**

作用：将模型数据通过页面展示给用户

## DispatcherServlet初始化过程

DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet 生命周期来进行调度。

a>初始化WebApplicationContext

b>创建WebApplicationContext

c>DispatcherServlet初始化策略

FrameworkServlet创建WebApplicationContext后，刷新容器，调用onRefresh(wac)，此方法在DispatcherServlet中进行了重写，调用了initStrategies(context)方法，初始化策略，即初始化DispatcherServlet的各个组件

## SpringMVC的执行流程

1) 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获。

2) DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI），判断请求URI对应的映射：

a) 不存在

i. 再判断是否配置了mvc:default-servlet-handler

ii. 如果没配置，则控制台报映射查找不到，客户端展示404错误

iii. 如果有配置，则访问目标资源（一般为静态资源，如：JS,CSS,HTML），找不到客户端也会展示404错误

b) 存在则执行下面的流程

3) 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain执行链对象的形式返回。

4) DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。

5) 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(…)方法【正向】

6) 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

a) HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息

b) 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等

c) 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等

d) 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中

7) Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。

8) 此时将开始执行拦截器的postHandle(...)方法【逆向】。

9) 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。

10) 渲染视图完毕执行拦截器的afterCompletion(…)方法【逆向】。

11) 将渲染结果返回给客户端。
