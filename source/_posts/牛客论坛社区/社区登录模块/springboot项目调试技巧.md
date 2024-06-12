---
title: springboot项目调试
categories:
- nowcoder
tags:
- forum
- nowcoder
---
<meta name="referrer" content="no-referrer"/>

## 内容

**本节包括：**

- 日志类

<!--more-->

## Logger

采用`org.slf4j.Logger`

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = CommunityApplication.class)
public class LoggerTests {
    private static final Logger logger = LoggerFactory.getLogger(LoggerTests.class);

    @Test
    public void testLogger() {
        System.out.println(logger.getName());

        logger.debug("debug log");
        logger.info("info log");
        logger.warn("warn log");
        logger.error("error log");
    }
}
~~~

## Logger配置

不推荐，没有分类，也没有考虑日志文件过大问题

~~~java
logging.file.name=C:/data/tmpDown/community.log
~~~

**推荐：**

`logback-spirng.xml`放在`resource`目录下

超过5M会创建新的日志文件，超过30天删除

~~~xml
<!-- error file -->
<appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_PATH}/${APPDIR}/log_error.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOG_PATH}/${APPDIR}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <maxFileSize>5MB</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
        <maxHistory>30</maxHistory>
    </rollingPolicy>
    <append>true</append>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%d %level [%thread] %logger{10} [%file:%line] %msg%n</pattern>
        <charset>utf-8</charset>
    </encoder>
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>error</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
</appender>
~~~

只需要该模板的几处

~~~xml
<contextName>community</contextName>
<property name="LOG_PATH" value="C:\data\tmpDown"/>
<property name="APPDIR" value="community"/>
~~~

~~~xml
<logger name="com.hdb.community" level="debug"/>
~~~



