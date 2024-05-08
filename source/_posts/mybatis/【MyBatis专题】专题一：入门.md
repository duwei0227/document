---
layout: post
title: 【MyBatis专题】专题一：入门
categories: [mybatis]
description: 【MyBatis专题】专题一：入门
permalink: mybatis/introduce.html
date: 2024-05-08 13:57:09
tags:
---
本篇介绍`MyBatis`入门所需知识，包括`Maven`依赖，`Mapper`文件的执行等基础知识。

<!--more-->


### 一、引入MyBatis依赖

当前最新版本：**3.5.13**

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```



### 二、从XML文件中构建SqlSessionFactory

每个基于 `MyBatis` 的应用都是以一个 `SqlSessionFactory` 的实例为核心的。`SqlSessionFactory` 的实例可以通过 `SqlSessionFactoryBuilder` 获得。而 `SqlSessionFactoryBuilder` 则可以从 `XML` 配置文件或一个预先配置的 `Configuration` 实例来构建出 `SqlSessionFactory` 实例。

#### 1、mybatis全局配置文件

在`resources`目录创建`mybatis-config.xml`文件，初始添加如下内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties>
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mapper/StudentMapper.xml"/>
    </mappers>
</configuration>
```

关于`mybatis-config.xml`文件的详细介绍放在后文中。



#### 2、用于数据查询的Mapper文件

在`resource`目录下创建 `mapper`目录用于存放`mapper`文件，后续所有涉及到`mapper`的文件都应该放在此处目录，此处示例创建`StudentMapper.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.probiecoder.mapper.StudentMapper">
    <select id="selectByName" resultType="cn.probiecoder.entity.Student">
        select * from student where name = #{name}
    </select>
</mapper>
```



#### 4、使用mybatis-config.xml创建SqlSessionFactory

```java
String resource = "mybatis-config.xml";
        try (InputStream is = Resources.getResourceAsStream(resource)) {
            // 此处SqlSessionFactory应该使用单例模式保证全局唯一，且只需要获取一次即可  
            SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
            try (SqlSession sqlSession = factory.openSession();) {
                StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
                String name = "王麻子";
                Student student = studentMapper.selectByName(name);
                System.out.println(student);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```



#### 5、作用域和生命周期

**SqlSessionFactoryBuilder**

这个类可以被实例化、使用和丢弃，一旦创建了 `SqlSessionFactory`，就不再需要它了。 因此 `SqlSessionFactoryBuilder` 实例的最佳作用域是方法作用域（也就是局部方法变量）。 可以重用 `SqlSessionFactoryBuilder` 来创建多个 `SqlSessionFactory` 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

**SqlSessionFactory**

`SqlSessionFactory` 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 `SqlSessionFactory` 的最佳实践是在应用运行期间不要重复创建多次，多次重建 `SqlSessionFactory` 被视为一种代码“坏习惯”。因此 `SqlSessionFactory` 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

**SqlSession**

每个线程都应该有它自己的 `SqlSession` 实例。`SqlSession` 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 `SqlSession` 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 `SqlSession` 实例的引用放在任何类型的托管作用域中，比如 `Servlet` 框架中的 `HttpSession`。 如果你现在正在使用一种 Web 框架，考虑将 `SqlSession` 放在一个和 `HTTP` 请求相似的作用域中。 换句话说，每次收到 `HTTP` 请求，就可以打开一个 `SqlSession`，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 `finally` 块中。 下面的示例就是一个确保 `SqlSession` 关闭的标准模式：

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

在所有代码中都遵循这种使用模式，可以保证所有数据库资源都能被正确地关闭。



 **提示：对象生命周期和依赖注入框架**

依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。 例如：MyBatis-Spring

测试用数据表：

```sql
create table student(
    id bigint primary key auto_increment,
    name varchar(100) not null comment '名称',
    age tinyint not null comment '年龄',
    sex char(1) not null comment '性别 M:male F:female',
    creation_date datetime not null comment '创建日期'
);

insert into student (name,age,sex,creation_date) values('张三', 6, 'M', '2023-10-12 10:10:10');
insert into student (name,age,sex,creation_date) values('李四', 7, 'M', '2023-10-12 11:10:10');
insert into student (name,age,sex,creation_date) values('王麻子', 8, 'F', '2023-10-12 12:10:10');
```

知识来源：[https://mybatis.org/mybatis-3/zh/getting-started.html](https://mybatis.org/mybatis-3/zh/getting-started.html)



