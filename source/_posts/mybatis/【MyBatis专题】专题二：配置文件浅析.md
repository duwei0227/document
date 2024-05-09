---
layout: post
title: 【MyBatis专题】专题二：配置文件浅析
categories: [mybatis]
permalink: mybatis/setting.html
date: 2024-05-08 13:56:29
---
本节介绍`MyBatis`中一个很重要的配置文件，此配置文件的内容会影响`MyBatis`行为的设置和属性信息。

<!--more-->

顶层结构配置如下：

* `configuration`（配置）
  * [`properties`](#properties)（属性）
  * [`typeAliases`](#typeAliases)（类型别名）
  * [`typeHandlers`](#typeHandlers)（类型处理器）
  * [`objectFactory`](#objectFactory)（对象工厂）
  * [`plugins`](#plugins)（插件）



### <a id="properties">一、属性</a>

属性一般用于不同环境有不同配置的场景，例如数据库地址、用户名等信息。

语法：

```xml
<properties resource="" url="">
    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</properties>
```

`properties`可以通过`resource`或者`url`加载外部配置文件，也可以直接在`properties`节点追加`property`节点设置属性。



用法：

```xml
<dataSource type="POOLED">
  <property name="username" value="${username:root}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

 从 `MyBatis 3.4.2` 开始，可以通过占位符指定一个默认值；默认情况，此特性是关闭状态未开启，需要通过配置：`<property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/>` 启用默认值特性。



属性可以在`properties`节点、外部资源文件和方法参数不同位置指定，`MyBatis`按照下面的顺序加载：

- 首先读取在 `properties` 元素体内指定的属性。
- 然后根据 `properties` 元素中的 `resource` 属性读取类路径下属性文件，或根据 `url` 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
- 最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。



**注意事项**

如果在属性名中使用了 `":"` 字符（如：`db:username`），或者在 SQL 映射中使用了 OGNL 表达式的三元运算符（如： `${tableName != null ? tableName : 'global_constants'}`），就需要设置特定的属性来修改分隔属性名和默认值的字符。

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- 修改默认值的分隔符 -->
</properties>


<dataSource type="POOLED">
  <property name="username" value="${db:username?:ut_user}"/>
</dataSource>
```


<a id="typeAliases">二、类型别名（typeAliases）</a>

类型别名可为 `Java` 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

语法：

1、使用`typeAlias`对每一个实体类进行单独配置

```xml
<typeAliases>
    <typeAlias type="cn.probiecoder.entity.Student" alias="Student" />
</typeAliases>
```



2、使用 `package` 指定包名，`MyBatis` 会在包名下面搜索需要的 `Java Bean`;在没有注解的情况下，会使用`Bean`首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`；若有注解，则别名为其注解值。

```xml
<typeAliases>
    <package name="cn.probiecoder.entity"/>
</typeAliases>
```

```java
@Alias("Author")
public class Author {
    ...
}
```



**示例：**

在不指定类型别名的情况下，在`Mapper`文件中直接使用缩写别名会报如下错误：

```java
### The error may exist in mapper/StudentMapper.xml
### Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. The XML location is 'mapper/StudentMapper.xml'. Cause: org.apache.ibatis.builder.BuilderException: Error resolving class. Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'Student'.  Cause: java.lang.ClassNotFoundException: Cannot find class: Student
	at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
	at org.apache.ibatis.session.SqlSessionFactoryBuilder.build(SqlSessionFactoryBuilder.java:82)
	at org.apache.ibatis.session.SqlSessionFactoryBuilder.build(SqlSessionFactoryBuilder.java:66)
	at cn.probiecoder.config.SqlSessionFactoryBuildByXML.main(SqlSessionFactoryBuildByXML.java:18)
```



通过`typeAlias`设定类型别名

```xml
<typeAliases>
    <typeAlias type="cn.probiecoder.entity.Student" alias="Student" />
</typeAliases>

<select id="selectByName" resultType="Student">
    select * from student where name = #{name}
</select>
```



通过`package`设定类型别名

```xml
<typeAliases>
    <package name="cn.probiecoder.entity"/>
</typeAliases>

<select id="selectByName" resultType="Student">
    select * from student where name = #{name}
</select>
```



<a id="typeHandlers">三、类型处理器（typeHandlers）</a>

`MyBatis` 在设置预处理语句（`PreparedStatement`）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 `Java` 类型。



**自定义类型处理**

* 实现 `org.apache.ibatis.type.TypeHandler` 接口
* 继承`org.apache.ibatis.type.BaseTypeHandler`

`Java`类型绑定：

- 在类型处理器的配置元素（typeHandler 元素）上增加一个 `javaType` 属性（比如：`javaType="String"`）；
- 在类型处理器的类上增加一个 `@MappedTypes` 注解指定与其关联的 Java 类型列表。 如果在 `javaType` 属性中也同时指定，则注解上的配置将被忽略。

`JDBC`类型绑定：

- 在类型处理器的配置元素上增加一个 `jdbcType` 属性（比如：`jdbcType="VARCHAR"`）；
- 在类型处理器的类上增加一个 `@MappedJdbcTypes` 注解指定与其关联的 JDBC 类型列表。 如果在 `jdbcType` 属性中也同时指定，则注解上的配置将被忽略。

**示例：**

自定义类型处理器类：

```java
package cn.probiecoder.config;

import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class VarcharTypeHandler implements TypeHandler<String> {
    @Override
    public void setParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter);
    }

    @Override
    public String getResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getString(columnName) + "columnName";
    }

    @Override
    public String getResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getString(columnIndex) + columnIndex;
    }

    @Override
    public String getResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getString(columnIndex) + columnIndex + "cs";
    }
}

```



`XML`配置文件，声明自定义类型处理器(此处也可以通过 `package` 指定目录)：

```xml
<typeHandlers>
    <!--        <package name="cn.probiecoder"/>-->
    <typeHandler handler="cn.probiecoder.config.VarcharTypeHandler" jdbcType="VARCHAR" javaType="String"/>
</typeHandlers>
```



执行结果：

由于字段`name`的类型为 `VARCHAR`，经过自定义类型转换后，拼接了`columnName`

```
Student(id=3, name=王麻子columnName, age=8, sex=F, creationDate=2023-10-12T12:10:10)
```



**枚举类型**

若想映射枚举类型 `Enum`，则需要从 `EnumTypeHandler` 或者 `EnumOrdinalTypeHandler` 中选择一个来使用。

**注意 `EnumTypeHandler` 在某种意义上来说是比较特别的，其它的处理器只针对某个特定的类，而它不同，它会处理任意继承了 `Enum` 的类。**

通过源码分析，可以得到`EnumTypeHandler`用于处理字符串枚举值，`EnumOrdinalTypeHandler`用于处理`int`类型的枚举值。



自定义字符串枚举类：

**注意事项：**

* 使用自定义枚举类转换时，需要在`mappers`文件中使用`resultMap`显示设置`typeHandlers`为`org.apache.ibatis.type.EnumTypeHandler`
* 枚举类的显示名需要和数据库值保持一致
* 在`config`文件中声明`typeHandlers`



示例：

枚举类

```java
package cn.probiecoder.enums;

public enum Sex {
    M("M"),
    F("F"),
    ;

    private final String code;

    Sex(String code) {
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}

```



`mappers`：

```xml
<resultMap id="resultMap" type="Student">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
</resultMap>
```



`config`配置文件：

```xml
<typeHandlers>
<!--        <package name="cn.probiecoder"/>-->
    <typeHandler handler="cn.probiecoder.config.VarcharTypeHandler" jdbcType="VARCHAR" javaType="String"/>
    <typeHandler handler="org.apache.ibatis.type.EnumTypeHandler" javaType="cn.probiecoder.enums.Sex"/>
</typeHandlers>
```



<a id="objectFactory">四、对象工厂（objectFactory）</a>

`MyBatis`创建结果对象的实例时，会使用一个对象工厂(`ObjectFactory`)实例来完成实例化工作。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。如果想覆盖对象工厂的默认行为，可以通过继承`DefaultObjectFactory`创建自己的对象工厂。



```java
package cn.probiecoder.config;

import org.apache.ibatis.reflection.factory.DefaultObjectFactory;

import java.util.List;
import java.util.Properties;

public class CustomerObjectFactory extends DefaultObjectFactory {

    @Override
    public <T> T create(Class<T> type) {
        System.out.println("待创建数据类型：" + type.getCanonicalName());
        return super.create(type);
    }

    @Override
    public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
        return super.create(type, constructorArgTypes, constructorArgs);
    }

    @Override
    protected Class<?> resolveInterface(Class<?> type) {
        return super.resolveInterface(type);
    }

    @Override
    public <T> boolean isCollection(Class<T> type) {
        return super.isCollection(type);
    }

    @Override
    public void setProperties(Properties properties) {
        System.out.println("config配置属性 isMybatis 值：" + properties.getProperty("isMybatis"));
        super.setProperties(properties);
    }
}

```



```xml
<objectFactory type="cn.probiecoder.config.CustomerObjectFactory">
    <property name="isMybatis" value="true"/>
</objectFactory>
```



执行结果：

```
config配置属性 isMybatis 值：true
待创建数据类型：java.util.List
Student(id=3, name=王麻子, age=8, sex=F, creationDate=2023-10-12T12:10:10)
```



<a id="plugins">五、插件（plugins）</a>

`MyBatis` 允许在映射语句执行过程中的某一点进行拦截调用。默认情况下，`MyBatis` 允许使用插件来拦截的方法调用包括：

- 接口：`org.apache.ibatis.executor.Executor`
  - 拦截方法：`update, query, flushStatements, commit, rollback, getTransaction, close, isClosed`
- 接口：`ParameterHandler`
  - 拦截方法：`org.apache.ibatis.executor.parameter.getParameterObject, setParameters`
- 接口：`ResultSetHandler`
  - 拦截方法：`org.apache.ibatis.executor.resultset.handleResultSets, handleOutputParameters`
- 接口：`StatementHandler`
  - 拦截方法：`org.apache.ibatis.executor.statement.prepare, parameterize, batch, update, query`



插件的使用只需要实现 `Interceptor` 接口，并指定想要拦截的方法签名即可。

**注意：通过插件修改已有方法的逻辑时，可能会破坏`MyBatis`的核心模块，所以使用时需要特别当心。**



自定义插件方式如下：

* 继承 `Interceptor`
* 增加注解 `@Intercepts({@Signature(type=XXX.class, method="要拦截的接口方法", args={拦截方法入参类型})})`



```java
package cn.probiecoder.config;

import org.apache.ibatis.executor.parameter.ParameterHandler;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Signature;

import java.sql.PreparedStatement;
import java.util.Properties;

@Intercepts({
        @Signature(
                type = ParameterHandler.class,
                method = "setParameters",
                args = {PreparedStatement.class}
        )
})
public class ParameterHandlerPlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("结果执行前参数设置");
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        return Interceptor.super.plugin(target);
    }

    @Override
    public void setProperties(Properties properties) {
        Interceptor.super.setProperties(properties);
    }
}

```

