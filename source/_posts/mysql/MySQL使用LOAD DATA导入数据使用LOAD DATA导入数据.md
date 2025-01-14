---
layout: post
title: MySQL使用LOAD DATA导入数据
categories: [MySQL]
permalink: mysql/load_data.html
---

### 

**基础语法：**

```mysql
LOAD DATA
    [LOW_PRIORITY | CONCURRENT] [LOCAL]
    INFILE 'file_name'
    [REPLACE | IGNORE]
    INTO TABLE tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [CHARACTER SET charset_name]
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [IGNORE number {LINES | ROWS}]
    [(col_name_or_user_var
        [, col_name_or_user_var] ...)]
    [SET col_name={expr | DEFAULT}
        [, col_name={expr | DEFAULT}] ...]
```



* `Local`: 从本地客户端读取文件，需要服务端和客户端同时开发权限，设置`local_infile=ON`
* `REPLACE`: 使用文件`key`匹配的行替换数据库中数据，执行结果表现为`deleted xx`
* `IGNORE`:忽略主键冲突的数据，如果使用`Local`时，未指定`REPLACE`和`IGNORE`的时候，效果等同于 `IGNORE`
* `TERMINATED BY`:定义属性`FIELDS`或行`LINES`终止字符，例如： `\n`
* `ENCLOSED BY`:定义属性`FIELDS`被什么字符包括，例如：`"张三"`解析以后只存储`张三`
* `IGNORE`:忽略`number`行，如果输入文件存在表头时，可以通过`IGNORE`自动忽略
* `SET`对输入内容进行格式转换处理，例如日期字符串格式化转换
  * 在字段行定义变量，例如 @timestamp
  * `SET`位置引用变量，并设置字段值， `SET birtyday = STR_TO_DATE(@timestamp, '%Y-%m-%d %H:%i:%s')`



**使用示例：** 

```mysql
create table load_data_test(
	id int primary key,
    name varchar(20),
    birthday datetime
);
```



```csv
id,name,birthday
4,"张三",1991-01-17 19:18:17
7,"李四",1993-11-11 13:14:15
```



```mysql
load data local 
infile '/home/duwei/workspace/load_data.csv' 
REPLACE 
into table learn.load_data_test 
FIELDS TERMINATED BY ','  ENCLOSED BY '"'  
LINES TERMINATED BY '\n'  
IGNORE 1 LINES 
(id, name, @timestamp) 
SET birthday = STR_TO_DATE(@timestamp, '%Y-%m-%d %H:%i:%s');
```





**问题：**

1、`ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement`

问题原因：

从`MySQL`服务器读取文件，文件不在`MySQL`服务允许的路径，可以通过 `show variables like 'secure_file_priv';`查看路径



解决方式：

* 将文件转移到`MySQL`服务权限路径下执行
* 禁用`secure_file_priv`,在`/etc/my.cnf`文件中添加`secure_file_priv=''`(在`[mysqld]`下)
* 自定义`secure_file_priv`**文件路径**，自定义的路径需要对`mysql`用户授予权限，否则在服务启动或者`LOAD DATA`时提示`ERROR 13 (HY000)`



2、`ERROR 3948 (42000): Loading local data is disabled; this must be enabled on both the client and server sides`

问题原因：

使用`LOAD DATA`时，服务端和客户端未授予相关权限



解决方式：

* 服务端：
  * 会话级：`set global local_infile='ON';`
  * 永久生效：`/etc/my.cnf --> [mysqld] --> local_infile=ON`
* 客户端：
  * `mysql --local-infile=1`



**参考文档：**

[https://dev.mysql.com/doc/refman/8.0/en/load-data.html](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)

[https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_secure_file_priv)



