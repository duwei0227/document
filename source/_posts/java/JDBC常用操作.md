---
layout: post
title: JDBC常用操作
categories: [Java]
description: JDBC常用操作
permalink: java/jdbc.html
date: 2024-05-08 14:02:10
tags:
---
本文介绍JDBC相关操作


<!-- more -->


### 一、查询条件中使用IN
可以使用`JDBC`提供的`setArray`方法设置（此处由于无`Oracle`数据库暂未验证）
示例：
```java
String sql = "select * from table_name where id in ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setArray(1, conn.createArrayOf("BIGINT", instanceIdList.toArray()));
ps.executeQuery();
```

在 `MySQL`环境下，执行 `setArray`会抛出`java.sql.SQLFeatureNotSupportedException`异常，需要自行进行处理。
个人处理方案如下(对于查询字段为数字类型时，拼接单引号`'`不会导致索引失效)
```java
private static String forEach(List<String> idList) {
    StringBuilder builder = new StringBuilder("(");
    idList.forEach(id -> {
        builder.append("'").append(id).append("'").append(",");
    });
    builder.deleteCharAt(builder.length() - 1);
    builder.append(")");
    return builder.toString();
}
```
