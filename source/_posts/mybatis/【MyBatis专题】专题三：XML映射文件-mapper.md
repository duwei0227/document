---
layout: post
title: 【MyBatis专题】专题三：XML映射文件(mapper)
categories: [Mybatis]
permalink: mybatis/mapper.html
date: 2024-05-08 13:55:53
---


一个空的`Mapper`文件结构如下，需要指定命名空间，且命名空间需要**唯一**，命名空间会和`Java`接口绑定，一般会添加注解`@Mapper`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.probiecoder.mapper.StudentMapper">
</mapper>
```



`SQL`映射文件支持的顶级节点元素(按照应被定义的顺序列出):

* `cache`-命名空间的缓存配置
* `cache-ref`– 引用其它命名空间的缓存配置
* `resultMap` - 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
* `sql`– 可被其它语句引用的可重用语句块。
* `insert`– 映射插入语句
* `update`– 映射更新语句
* `delete`– 映射删除语句
* `select`– 映射查询语句



### 一、select

一个简单的常用的`select`配置如下：

```xml
<select id="" parameterType="" resultType="" resultMap="" >
    select * from student where name = #{name}
</select>
```

完整的属性配置和介绍：

| 属性            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `id`            | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType` | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以根据语句中实际传入的参数计算出应该使用的类型处理器（TypeHandler），默认值为未设置（unset）。 |
| `resultType`    | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。 |
| `resultMap`     | 对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。 |
| `flushCache`    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| `useCache`      | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
| `timeout`       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| `fetchSize`     | 这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。 |
| `statementType` | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `resultSetType` | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。 |
| `resultOrdered` | 这个设置仅针对嵌套结果 select 语句：如果为 true，则假设结果集以正确顺序（排序后）执行映射，当返回新的主结果行时，将不再发生对以前结果行的引用。 这样可以减少内存消耗。默认值：`false`。 |
| `resultSets`    | 这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。 |



### 二、insert、update、delete

对于`insert`、`update`、`delete`一般使用中只需要指定id即可

```xml
<insert id="">
    SQL语句
</insert>

<update id="">
    SQL语句
</update>

<delete id="">
	SQL语句
</delete>
```

支持的属性配置如下：

| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `id`               | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType`    | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以根据语句中实际传入的参数计算出应该使用的类型处理器（TypeHandler），默认值为未设置（unset）。 |
| `flushCache`       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。 |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| `statementType`    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `useGeneratedKeys` | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。 |
| `keyProperty`      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `keyColumn`        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |



### sql

定义可重用的 SQL 代码片段，以便在其它语句中使用。

```xml
<sql id="resultColumn">
    select id, name, age,sex,creation_date
</sql>
```

在`select`、`update`、`delete`、`inserst`中使用`<include refid="">`引用

```xml
<select id="selectByName" resultMap="resultMap">
    <include refid="resultColumn" />
    from student where name = #{name}
</select>
```



### 结果映射(resultMap)

相比与`ResultType`结果映射来说，`ResultMap`拥有更强的表现能力。

支持的节点配置：

* `constructor` - 用于在实例化类时，注入结果到构造方法中

  * `idArg` - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能
  * `arg` - 将被注入到构造方法的一个普通结果

* `id` – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能

* `result` – 注入到字段或 JavaBean 属性的普通结果

* `association` – 一个复杂类型的关联；许多结果将包装成这种类型

  * 嵌套结果映射 – 关联可以是 `resultMap` 元素，或是对其它结果映射的引用

* `collection` – 一个复杂类型的集合z

  * 嵌套结果映射 – 集合可以是 `resultMap` 元素，或是对其它结果映射的引用

* `discriminator` – 使用结果值来决定使用哪个`resultMap`

  * `case` – 基于某些值的结果映射
    * 嵌套结果映射 – `case` 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射

  

  ​									`ResultMap` 的属性列表

| 属性          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `id`          | 当前命名空间中的一个唯一标识，用于标识一个结果映射。         |
| `type`        | 类的完全限定名, 或者一个类型别名（关于内置的类型别名，可以参考上面的表格）。 |
| `autoMapping` | 如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。默认值：未设置（unset）。 |



#### 1、id & result

```xml
<id property="id" column="post_id"/>
<result property="subject" column="post_subject"/>
```

​	

​												Id 和 Result 的属性

| 属性          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `property`    | 映射到列结果的字段或属性。如果 JavaBean 有这个名字的属性（property），会先使用该属性。否则 MyBatis 将会寻找给定名称的字段（field）。 无论是哪一种情形，你都可以使用常见的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。 |
| `column`      | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 `resultSet.getString(columnName)` 方法的参数一样。 |
| `javaType`    | 一个 Java 类的全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。 |
| `jdbcType`    | JDBC 类型，所支持的 JDBC 类型参见这个表格之后的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可以为空值的列指定这个类型。 |
| `typeHandler` | 我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的全限定名，或者是类型别名。 |



示例：

```xml
<resultMap id="resultMap" type="Student">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
</resultMap>
```



#### 2、constructor

构造方法注入允许在初始化时为类设置属性的值，而不用暴露出公有方法。

```xml
<constructor>
   <idArg column="id" javaType="int"/>
   <arg column="username" javaType="String"/>
   <arg column="age" javaType="_int"/>
</constructor>
```



#### 3、关联 association

关联一般为主对象中某个属性也是一个对象。

关联的不同之处是，你需要告诉 MyBatis 如何加载关联。MyBatis 有两种不同的方式加载关联：

- 嵌套 Select 查询：通过执行另外一个 SQL 映射语句来加载期望的复杂类型。
- 嵌套结果映射：使用嵌套的结果映射来处理连接结果的重复子集。



| 属性          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `property`    | 映射到列结果的字段或属性。如果用来匹配的 JavaBean 存在给定名字的属性，那么它将会被使用。否则 MyBatis 将会寻找给定名称的字段。 无论是哪一种情形，你都可以使用通常的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。 |
| `javaType`    | 一个 Java 类的完全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。 |
| `jdbcType`    | JDBC 类型，所支持的 JDBC 类型参见这个表格之前的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可能存在空值的列指定这个类型。 |
| `typeHandler` | 我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的完全限定名，或者是类型别名。 |

##### 关联的嵌套结果映射

| 属性            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `resultMap`     | 结果映射的 ID，可以将此关联的嵌套结果集映射到一个合适的对象树中。 它可以作为使用额外 select 语句的替代方案。它可以将多表连接操作的结果映射成一个单一的 `ResultSet`。这样的 `ResultSet` 有部分数据是重复的。 为了将结果集正确地映射到嵌套的对象树中, MyBatis 允许你“串联”结果映射，以便解决嵌套结果集的问题。使用嵌套结果映射的一个例子在表格以后。 |
| `columnPrefix`  | 当连接多个表时，你可能会不得不使用列别名来避免在 `ResultSet` 中产生重复的列名。指定 columnPrefix 列名前缀允许你将带有这些前缀的列映射到一个外部的结果映射中。 详细说明请参考后面的例子。 |
| `notNullColumn` | 默认情况下，在至少一个被映射到属性的列不为空时，子对象才会被创建。 你可以在这个属性上指定非空的列来改变默认行为，指定后，Mybatis 将只在这些列中任意一列非空时才创建一个子对象。可以使用逗号分隔来指定多个列。默认值：未设置（unset）。 |
| `autoMapping`   | 如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。注意，本属性对外部的结果映射无效，所以不能搭配 `select` 或 `resultMap` 元素使用。默认值：未设置（unset）。 |

示例：

```java
public class Grade {
    private Long id;
    private String name;
}


public class Student {
    private Long id;
    private String name;
    private Integer age;
    private Sex sex;
    private Grade grade;
}
```



```xml
<select id="selectByName" resultMap="resultMap">
    select s.id, s.name, s.age, s.sex, g.id g_id, g.name g_name
    from student s left join grade g on s.grade_id = g.id where s.name = #{name}
</select>

<resultMap id="resultMap" type="Student">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="age" column="age" />
    <result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
    <association property="grade" column="grade" resultMap="gradeResultMap" columnPrefix="g_"/>
</resultMap>

<resultMap id="gradeResultMap" type="cn.probiecoder.entity.Grade">
    <id property="id" column="id" />
    <result property="name" column="name" />
</resultMap>
```



#### 4、集合 collection

集合类似与关联`association`，只是关系表现为：一对多的关系

```java
public class Teacher {
    private Long id;
    private String name;
    private List<Student> students;
}


public class Student {
    private Long id;
    private String name;
    private Integer age;
    private Sex sex;
}
```



联表查询语句：

```xml
<select id="selectStudentByTeacherId" resultMap="teachResultMap">
    select t.id t_id, t.name t_name,s.id s_id, s.name s_name,s.age s_age,
    s.sex s_sex, s.creation_date s_creation_date,s.teacher_id s_teacher_id
    from teacher t left outer join student s on t.id = s.teacher_id where t.id = #{teacherId};
</select>
```

结果集映射：

```xml
<resultMap id="resultMap" type="Student">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="age" column="age" />
    <result property="sex" column="sex" typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
</resultMap>

<resultMap id="teachResultMap" type="cn.probiecoder.entity.Teacher">
    <id property="id" column="t_id" />
    <result property="name" column="t_name" />
    <collection property="students" ofType="Student" resultMap="resultMap" columnPrefix="s_" />
</resultMap>
```

* `ofType`指定列表中元素的类型
* `resultMap`复用已有的结果集映射`map`，也可以直接在`collection`节点下设置结果字段映射关系
* `columnPrefix`联表查询时，不同表的字段会相同，为了区分会增加前缀，通过`columnPrefix`在映射到关联数据集时会自动处理前缀，实现关联表结果集映射复用

来源：https://mybatis.org/mybatis-3/zh/sqlmap-xml.html