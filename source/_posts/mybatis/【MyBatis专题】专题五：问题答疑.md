---
layout: post
title: 【MyBatis专题】专题五：问题答疑
categories: [mybatis]
description: 【MyBatis专题】专题五：问题答疑
permalink: mybatis/problems.html
date: 2024-05-08 13:48:05
tags:
---
本节针对日常使用`MyBatis`时遇到的相关问题进行记录，便于后续其他项目遇到可以及时解决问题。


<!--more-->

### 问题一：MyBatis日志配置

Mybatis 通过使用内置的日志工厂提供日志功能。内置日志工厂将会把日志工作委托给下面的实现之一：

- SLF4J
- Apache Commons Logging
- Log4j 2
- JDK logging

MyBatis 内置日志工厂基于运行时自省机制选择合适的日志工具。它会使用第一个查找得到的工具（按上文列举的顺序查找）。如果一个都未找到，日志功能就会被禁用。



如果类路径中存在多个符合条件的日志工具时，可以通过在`MyBatis` 配置文件 `mybatis-config.xml` 里面添加一项 `setting` 来选择别的日志工具。

```xml
<configuration>
  <settings>
    ...
    <setting name="logImpl" value="LOG4J"/>
    ...
  </settings>
</configuration>
```

`logImpl` 可选的值有：`SLF4J、LOG4J、LOG4J2、JDK_LOGGING、COMMONS_LOGGING、STDOUT_LOGGING、NO_LOGGING`，或者是实现了接口 `org.apache.ibatis.logging.Log` 的，且构造方法是以字符串为参数的类的完全限定名。



使用Log4j2配置日志示例：

`pom.xml`

```xml
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.x.x</version>
</dependency>
```

`log4j2.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns="http://logging.apache.org/log4j/2.0/config">

    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%5level [%t] - %msg%n"/>
        </Console>
    </Appenders>

    <Loggers>
        <Logger name="cn.probiecoder.mapper.StudentMapper" level="debug"/>
        <Root level="info" >
            <AppenderRef ref="stdout"/>
        </Root>
    </Loggers>

</Configuration>
```

日志知识来源：https://mybatis.org/mybatis-3/zh/logging.html

### 问题二、参数设置中 #{} 和 ${}的区别

* `#{}`会在`build`阶段将参数用占位符`?`替换，在`ParameterHandler.setParameters`阶段将实际的值进行设置，设置时会按照字段数据类型进行设置，即如果传入的参数值为一段SQL，由于数据类型为`VARCHAR`会被引号包括，失去`SQL`注入的能力。

  ```java
  TypeHandler typeHandler = parameterMapping.getTypeHandler();
  JdbcType jdbcType = parameterMapping.getJdbcType();
  if (value == null && jdbcType == null) {
    jdbcType = configuration.getJdbcTypeForNull();
  }
  try {
    typeHandler.setParameter(ps, i + 1, value, jdbcType);
  }
  ```

  那么`SQL`语句的参数在什么时候进行替换的呢？

  时间节点发生在`SqlSessionFactoryBuilder().build()`构造阶段，在处理`mapper`文件中各个模块时，会判断参数格式是否为 `#{}`形式，如果是会将对应参数替换为问号 `?`(`SqlSourceBuilder.parse(String originalSql, Class<?> parameterType, Map<String, Object> additionalParameters)`) 

  

* `${}`不会在`build`阶段进行替换，而是在实际调用`mapper`语句时，将`${}`替换为实际的数据值，替换发生在 `TextSqlNode.apply`处；替换时直接将参数用实际值替换，即如果参数值为 `or 1 = 1`的形式，最终形成的`SQL`就会包含这一段合法的`SQL`逻辑，进而产生`SQL`注入的问题



