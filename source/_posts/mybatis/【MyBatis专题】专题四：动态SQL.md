---
layout: post
title: 【MyBatis专题】专题四：动态SQL
categories: [mybatis]
description: 【MyBatis专题】专题四：动态SQL
permalink: mybatis/dynamic.html
date: 2024-05-08 13:55:10
tags:
---
动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。


<!--more-->


### if

根据`test`条件决定是否包含标签内语句

```xml
<if test="author != null and author.name != null">
    AND author_name like #{author.name}
</if>
```



### choose、when、otherwise

如果想从多个条件中选择一个使用可以考虑使用 `choose`、`when`、`otherwise`，语法类似于`Java`中的`switch`。

```xml
<choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
</choose>
```



### where

`<where>`标签解决的问题是当标签`if`所有条件都不满足或者中间部分条件满足时，自动去除子句中的`AND`或 `OR`或`where`，`<where>`素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。

例如如下示例，当`title`为空，`author`不为空时，不使用`<where>`标签时，`SQL`语句为：`select xxx from xxx where and author = ?`，很明显`where`子句中多了一个`and`语法错误；使用`<where>`标签时，`MyBatis`会自动移除子句中多余的`and`。

```xml
<where>
	<if test="title != null">
    	and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</where>
```



### trim

如果 *where* 元素与期望的不太一样时，也可以通过自定义 trim 元素来定制 *where* 元素的功能。

```xml
<trim prefix="SET" suffixOverrides=",">
    <if test="name != null">name = #{name},</if>
    <if test="age != null">age = #{age},</if>
</trim>
```

* `prefix`语句拼接前缀
* `prefixOverrides`移除指定前缀字符
* `suffix`语句拼接后缀字符
* `suffixOverrides`移除指定后缀字符



### set

`set`用于动态语句更新，`set`元素会动态地在行首插入 `SET` 关键字，并会删掉额外的逗号

```xml
<set>
    <if test="name != null">name = #{name},</if>
    <if test="age != null">age = #{age},</if>
</set>
```



### foreach

需要对一个集合进行遍历时可以使用`foreach`语法进行循环处理(常见与`IN`参数拼接)；允许指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量

```xml
<foreach item="item" index="index" collection="list"
        open="ID in (" separator="," close=")" nullable="true">
      #{item}
</foreach>
```

* `item`集合中的元素
* `index`元素索引位置
* `list`需要遍历的集合，支持`List`、`Map`、`Set`或数组，为`Map`时，`index`是键，`item`是值
* `open`拼接结果左边开始字符
* `close`拼接结果右边结束字符



来源：https://mybatis.org/mybatis-3/zh/dynamic-sql.html