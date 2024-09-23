---
layout: post
title: Spring Boot问题解决汇总
categories: [Spring]
permalink: spring/problems.html

---



### 一、`Invalid value type for attribute 'factoryBeanObjectType': java.lang.String`

```java

org.springframework.beans.factory.BeanDefinitionStoreException: Invalid bean definition with name 'userMapper' defined in file [shop-user/target/classes/cn/probiecoder/shopuser/mapper/UserMapper.class]: Invalid value type for attribute 'factoryBeanObjectType': java.lang.String
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getTypeForFactoryBean(AbstractAutowireCapableBeanFactory.java:857) ~[spring-beans-6.1.12.jar:6.1.12]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getType(AbstractBeanFactory.java:743) ~[spring-beans-6.1.12.jar:6.1.12]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAnnotationOnBean(DefaultListableBeanFactory.java:735) ~[spring-beans-6.1.12.jar:6.1.12]
	at org.springframework.boot.sql.init.dependency.AnnotationDependsOnDatabaseInitializationDetector.detect(AnnotationDependsOnDatabaseInitializationDetector.java:36) ~[spring-boot-3.3.3.jar:3.3.3]
	at 
```



升级`POM`依赖中`mybatis-spring`的版本高于`3.0.3`

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.3</version>
</dependency>
```



如果使用的是`mybatis-plus-boot-starter`需要先排除`mybatis-plus`引用的旧版本的`mybatis-spring`：`

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>${mybatis-plus.version}</version>
    <exclusions>
        <exclusion>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.0.4</version>
</dependency>
```



### 二 `java: java.lang.NoSuchFieldError: Class com.sun.tools.javac.tree.JCTree$JCImport does not have member field 'com.sun.tools.javac.tree.JCTree qualid'`

```java
java: java.lang.NoSuchFieldError: Class com.sun.tools.javac.tree.JCTree$JCImport does not have member field 'com.sun.tools.javac.tree.JCTree qualid'

```

问题原因：

`JDK21`版本要求的最低`lombok`版本为`1.18.30`，升级版本：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.34</version>
    <scope>provided</scope>
</dependency>
```

