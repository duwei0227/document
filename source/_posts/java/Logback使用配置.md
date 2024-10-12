---
layout: post
title: Logback使用配置
categories: [Java]
permalink: java/logback.html
---

### 一、Appender定义

#### 1、ConsoleAppender

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] [%thread] %logger{36} => %message %n</pattern>
    </layout>
</appender>
```



#### 2、RollingFileAppender

```xml
 <appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!-- 支持多个JVM同时写一个文件(容器部署，挂载共享磁盘) 不支持FixedWindowRollingPolicy -->
    <!--        <prudent>true</prudent>-->
    <!--活动文件-->
    <file>文件保存路径</file>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] [%thread] %logger{36} => %message %n</pattern>
        <charset>UTF-8</charset>
     </encoder>
</appender>
```



#### 3、AsyncAppender

```xml
<appender name="AsyncLogFile" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="fileLog" />
</appender>
```



### 二、文件滚动策略

#### 1、SizeAndTimeBasedRollingPolicy

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <!-- %i: 归档文件索引序号 .gz: logback 根据pattern后缀启用压缩-->
    <fileNamePattern>${TSF_LOG_DIR}/%d{yyyy-MM-dd,aux}/stdout-%d{yyyy-MM-dd HH}.%i.log.gz</fileNamePattern>
    <!-- 单个文件允许的最大值 -->
    <maxFileSize>100MB</maxFileSize>
    <!-- 归档文件保留的最大数量 -->
    <maxHistory>${MAX_RESERVE}</maxHistory>
    <!-- 限制所有存档日志文件总大小，超过默认删除最早的日志文件，重新生成新的日志文件，即 %i 的序号会持续增加 -->
    <!-- 与 maxHistory 结合使用，maxHistory 优先级高于 totalSizeCap -->
    <!-- logback 1.2 版本 totalSizeCap 删除索引超过1000的文件有bug -->
    <totalSizeCap>512MB</totalSizeCap>
    <cleanHistoryOnStart>true</cleanHistoryOnStart>
</rollingPolicy>
```



#### 2、FixedWindowRollingPolicy

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
    <!-- %i: 归档文件索引序号 .gz: logback 根据pattern后缀启用压缩-->
    <fileNamePattern>stdout.%i.log</fileNamePattern>
    <minIndex>1</minIndex>
    <maxIndex>4</maxIndex>
</rollingPolicy>
```



### 三、日志级别和输出位置设置

#### 1、全局默认

```xml
<root level="info">
    <appender-ref ref="AsyncLogFile" />
    <appender-ref ref="STDOUT" />
</root>
```



#### 2、指定类路径

```xml
<!-- 对特定类路径自定义日志级别以及日志输出位置 -->
<logger name="cn.probiecoder" level="info" additivity="false" includeLocation="true" >
    <appender-ref ref="AsyncLogFile" />
    <appender-ref ref="STDOUT" />
</logger>
```





### 四、在logback.xml中使用if条件

语法如下：

```xml
   <!-- if-then form -->
   <if condition="some conditional expression">
    <then>
      ...
    </then>
  </if>

  <!-- if-then-else form -->
  <if condition="some conditional expression">
    <then>
      ...
    </then>
    <else>
      ...
    </else>
  </if>
```

表达式：

* `property()`或`p()`：返回`property`的字符串值，如果`property`为定义，会返回空字符串(`""`)
* `isDefined`：判断`property`是否定义
* `isNull`：判断`property`是否为`null`



在使用`condition`前需要先引入如下依赖：

```xml
<dependency>
    <groupId>org.codehaus.janino</groupId>
    <artifactId>janino</artifactId>
    <version>3.1.12</version>
</dependency>
```



示例：

```xml
<if condition='isDefined("TSF_ENABLED")'>
    <then>
         <if condition="${TSF_ENABLED} == false}">
             <then>
                 <property name="CONSOLE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level [%thread] %logger{36} => %message %n"/>
                 <property name="FILE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] [%thread] %logger{36} => %message %n"/>
             </then>
             <else>
                 <property name="CONSOLE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
                 <property name="FILE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
             </else>
         </if>
    </then>
    <else>
        <property name="CONSOLE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
        <property name="FILE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
    </else>
</if>
```

通过`-D`指定`property`值

```shell
java -DFLAG=xxx -jar jar
```



### 五、配置示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- pattern不能缩略和完整混搭 -->
   <if condition='isDefined("ENABLED")'>
        <then>
             <if condition="${ENABLED} == false}">
                 <then>
                     <property name="CONSOLE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level [%thread] %logger{36} => %message %n"/>
                     <property name="FILE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] [%thread] %logger{36} => %message %n"/>
                 </then>
                 <else>
                     <property name="CONSOLE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
                     <property name="FILE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
                 </else>
             </if>
        </then>
        <else>
            <property name="CONSOLE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
            <property name="FILE_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss} [%level] %trace [%thread] %logger{36} => %message %n"/>
        </else>
    </if>

    <property name="CHARSET" value="UTF-8" />
    <property name="MAX_RESERVE" value="2"/>

    <!-- 定义所有的appender -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>${CONSOLE_PATTERN}</pattern>
        </layout>
    </appender>

    <appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 支持多个JVM同时写一个文件(容器部署，挂载共享磁盘) 不支持FixedWindowRollingPolicy -->
        <!--        <prudent>true</prudent>-->
        <!--活动文件-->
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_PATTERN}</pattern>
            <charset>${CHARSET}</charset>
        </encoder>

        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- %i: 归档文件索引序号 .gz: logback 根据pattern后缀启用压缩-->
            <fileNamePattern>${LOG_DIR}/%d{yyyy-MM-dd,aux}/stdout-%d{yyyy-MM-dd HH}.%i.log.gz</fileNamePattern>
            <!-- 单个文件允许的最大值 -->
            <maxFileSize>100MB</maxFileSize>
            <!-- 归档文件保留的最大数量 -->
            <maxHistory>${MAX_RESERVE}</maxHistory>
            <!-- 限制所有存档日志文件总大小，超过默认删除最早的日志文件，重新生成新的日志文件，即 %i 的序号会持续增加 -->
            <!-- 与 maxHistory 结合使用，maxHistory 优先级高于 totalSizeCap -->
            <!-- logback 1.2 版本 totalSizeCap 删除索引超过1000的文件有bug -->
            <totalSizeCap>512MB</totalSizeCap>
            <cleanHistoryOnStart>true</cleanHistoryOnStart>
        </rollingPolicy>
    </appender>

    <appender name="AsyncLogFile" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="fileLog" />
    </appender>

    <root level="info">
        <appender-ref ref="AsyncLogFile" />
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```



### 六、附录

`logback`依赖：

```xml
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.4.7</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.4.7</version>
</dependency>
```



参考：

https://janino-compiler.github.io/janino/

https://logback.qos.ch/manual/configuration.html#conditional