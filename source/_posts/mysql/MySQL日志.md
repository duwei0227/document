---
layout: post
title: MySQL日志
categories: [MySQL]
permalink: mysql/log.html
---





| Log Type          | Information Written to Log                                   |
| ----------------- | ------------------------------------------------------------ |
| Error log         | Problems encountered starting, running, or stopping          [**mysqld**](https://dev.mysql.com/doc/refman/8.4/en/mysqld.html) |
| General query log | 客户端执行的所有(成功)SQL记录                                |
| Binary log        | Statements that change data (also used for replication)      |
| Relay log         | Data changes received from a replication source server       |
| Slow query log    | 慢SQL记录，取决于 [`long_query_time`](https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html#sysvar_long_query_time) 系统变量 |
| DDL logs          | Atomic DDL operations performed by DDL statements            |





### 一、General log 和 Slow log



### 1、日志存储位置

支持如下存储方式：

* `FILE`：文件，默认存储方式
* `TABLE`: 数据表 
* `NONE`：不存储



### 1.1 查看当前日志存储方式

```mysql
show variable like 'log_output';
```

```mysql
mysql> show variables like 'log_output';
+---------------+------------+
| Variable_name | Value      |
+---------------+------------+
| log_output    | FILE       |
+---------------+------------+
1 row in set (0.00 sec)
```



### 1.2 全局修改日志存储方式

```mysql
set global log_output = 'TABLE,FILE';
```



### 1.3 设置持久化

通过`global`设置的系统变量，在`mysql`服务重启后会重新还原为默认值，需要在`my.cnf`中修改进行持久化

```cnf
[mysqld]
log_output=TABLE,FILE
```



### 2、General Log

记录客户端执行的所有SQL

### 2.1 查看当前日志记录开启以及文件保存位置

```mysql
show variables like 'general_log%';
```



**如果`log_output`支持`TABLE`时，日志信息会记录在 `mysql.general_log`数据表中**



### 2.2 日志文件日志内容

```sql
Time                 Id Command    Argument
2024-11-08T00:42:06.651126Z	   11 Query	select * from learn.shop
```



### 2.3 数据表日志内容

```mysql
mysql> select *, convert(argument using utf8) args from mysql.general_log \G

*************************** 1. row ***************************
  event_time: 2024-11-08 08:42:06.651126
   user_host: root[root] @ localhost []
   thread_id: 11
   server_id: 1
command_type: Query
    argument: 0x73656C656374202A2066726F6D206C6561726E2E73686F70
		args: select * from learn.shop

```

默认情况下，`general_log`数据表中存储`SQL`语句的字段(`argument`)进行了编码，需要通过`convert(argument using utf8)`函数进行解码



### 2.4 设置持久化

通过`global`设置的系统变量，在`mysql`服务重启后会重新还原为默认值，需要在`my.cnf`中修改进行持久化

```cnf
[mysqld]
general_log=ON
general_log_file=/var/lib/mysql/general_08.log
```

修改完成后通过`sudo systemctl restart mysqld`重启`mysql`服务让配置生效



### 3、Slow Query Log

记录执行时间超过`long_query_time`变量定义的值时会记录慢`SQL`日志



### 3.1 查看慢SQL日志开启以及文件保存位置

```mysql
show variables like 'slow_query_log%';
```



### 3.2 查看慢SQL时间阈值

```mysql
show variables like 'long_query_time';
```

```mysql
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.01 sec)
```



### 3.3 修改慢SQL时间阈值

```mysql
set global long_query_time=1;
```

<font color="red">通过`global`修改慢SQL阈值的需要连接重新建立后才生效</font>



| System Variable | `long_query_time` |
| --------------- | ----------------- |
| 生效范围        | Global, Session   |
| 动态修改        | Yes               |
| 数据类型        | Numeric           |
| 默认值          | `10`              |
| 最小值          | `0`               |
| 最大值          | `31536000`(365天) |
| 单位            | 秒                |



如果`log_output`支持`TABLE`时，日志信息会记录在 `mysql.slow_log`数据表中



### 3.4 慢SQL日志文件内容

```sql
# Time: 2024-11-08T00:34:07.522426Z
# User@Host: root[root] @ localhost []  Id:    11
# Query_time: 0.000312  Lock_time: 0.000004 Rows_sent: 7  Rows_examined: 7
SET timestamp=1731026047;
select * from learn.shop;
```



### 3.5 慢SQL数据表日志内容

```mysql
mysql> select *, convert(sql_text using utf8) sql_detail from slow_log \G
*************************** 1. row ***************************
    start_time: 2024-11-08 08:35:50.723019
     user_host: root[root] @ localhost []
    query_time: 00:00:00.000302
     lock_time: 00:00:00.000004
     rows_sent: 7
 rows_examined: 7
            db: mysql
last_insert_id: 0
     insert_id: 0
     server_id: 1
      sql_text: 0x73656C656374202A2066726F6D206C6561726E2E73686F70
     thread_id: 11
    sql_detail: select * from learn.shop

```



默认情况下，`slow_log`数据表中存储`SQL`语句的字段(`sql_text`)进行了编码，需要通过`convert(sql_text using utf8)`函数进行解码



### 3.6 设置持久化

通过`global`设置的系统变量，在`mysql`服务重启后会重新还原为默认值，需要在`my.cnf`中修改进行持久化

```cnf
[mysqld]
slow_query_log=ON
slow_query_log_file=/var/lib/mysql/slow_08.log
long_query_time=0

```

修改完成后通过`sudo systemctl restart mysqld`重启`mysql`服务让配置生效



## 二、Bin Log



默认启用 `log_bin=[base_name]`系统变量，不指定默认为`binlog`



新文件创建：

* 服务启动或者重启
* 服务刷新日志
* 日志文件达到`max_binlog_size`设置（采用大事物时，文件大小可能会超过`max_binlog_size`的限制）



`mysqld`采用索引文件记录当前使用的`binlog`文件，默认情况和`binlog`保持一直的文件名(扩展名为`.index`)，可以通过 `--log-bin-index=[file_name]`命令选项修改



`binlog`文件和`binlog`索引文件默认存储在数据目录中，可以通过`--log-bin`选项指定为其他目录





文件加密

