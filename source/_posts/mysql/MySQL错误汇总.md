---
layout: post
title: MySQL错误汇总
categories: [MySQL]
permalink: mysql/errors.html
date: 2024-05-08 14:00:45
---


### 一、ERROR 1118 (42000): Row size too large

**问题背景**：修改表字段(数据类型为`varchar`)长度时提示1118

**问题原因**：`MySQL`对数据表`column`的数量以及行记录大小进行了限制，所以在创建数据表或者通过`alter`修改表结构时可能会触发1118错误

**解决方案**：保证行记录允许最大字段和在允许范围内，其中对于`BLOB`和`TEXT`分别占用9到12字节(bytes)



`MySQL`对于数据表允许最大行记录取决与以下几个因素：

* `MySQL`服务器本身允许的最大行限制为65535字节，即使存储引擎可以支持更大的行；其中对于`BLOB`和`TEXT`只占行记录的9到12字节，字段内容不计算在内

* 对于`InnoDB`引擎而言，数据存储按照数据页的形式，允许的最大行记录略小于数据页的一半(数据页通过参数`innodb_page_size`设置，支持`4KB、8KB、16KB、32KB`,默认为`16KB`)。例如，对于默认的`16KB InnoDB`页面大小，最大行大小略小于`8KB`。

  如果包含可变长度列的行超过了`InnoDB`的最大行大小，`InnoDB`会选择可变长度列用于外部页外存储，直到该行符合`InnoDB`的行大小限制。对于页外存储的可变长度列，本地存储的数据量因行格式而异。

  

  ```mysql
  mysql> show variables like 'innodb_page_size';
  +------------------+-------+
  | Variable_name    | Value |
  +------------------+-------+
  | innodb_page_size | 16384 |
  +------------------+-------+
  1 row in set (0.01 sec)
  
  ```

  

* 不同的数据存储格式(`storage format`)会包含不同数量的页头和尾部数据，这些信息也会影响行的可用存储；

  关于`InnoDB`行格式的介绍可以参考：https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html



示例：

```mysql
-- 建表时，超过允许最大行记录
create table max_row_size_test(
    id int primary key,
    name1 varchar(10000),
    name2 varchar(10000)
);


mysql> create table max_row_size_test(
    ->     id int primary key,
    ->     name1 varchar(10000),
    ->     name2 varchar(10000)
    -> );
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs


-- 修改表结构时超过允许最大行记录
create table max_row_size_test(
    id int primary key,
    name1 varchar(10000),
    name2 varchar(10)
);

mysql> create table max_row_size_test(
    ->     id int primary key,
    ->     name1 varchar(10000),
    ->     name2 varchar(10)
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> alter table max_row_size_test modify column name2 varchar(10000);
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs

```



参考来源：https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html