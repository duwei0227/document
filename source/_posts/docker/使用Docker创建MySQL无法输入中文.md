---
layout: post
title: 使用Docker创建MySQL无法输入中文
categories: [Docker]
description: 使用Docker创建MySQL无法输入中文
permalink: docker/mysql_chinese.html
date: 2024-05-08 14:03:59
tags:
excerpt: 在使用`docker`创建的`MySQL`容器中，使用终端连接数据后，无法输入中文或者中文查询结果为问号，本文介绍两种方式处理中文问题，方法一在重新进入容器后会失效，需要手工再次执行`source /etc/profile`，方法二永久生效。
---

### 方法一、在容器内修改系统字符编码

**进入容器后修改系统字符编码，在退出容器后，配置不在生效，需要重新进行 `source /etc/profile`**

1、进入mysql容器

```shell
docker exec -it mysql bash
```



2、查看系统字符编码

```shell
bash-4.4# locale
LANG=
LC_CTYPE="POSIX"
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=
```

当前字符集为 `POSIX`，不支持中文，需要修改系统字符集支持中文输入



3、查看系统支持字符集

```shell
bash-4.4# locale -a
C
C.utf8
POSIX
```

字符集列表中 `C.utf8`支持中文，设置系统字符集为 `C.utf8`



4、修改系统字符集

```shell
echo "export LANG=C.utf8" >> /etc/profile && source /etc/profile
```



5、再次进行数据操作，中文操作无问题

```sql
mysql> select * from student;
+----+-----------+-----+-----+---------------------+
| id | name      | age | sex | creation_date       |
+----+-----------+-----+-----+---------------------+
|  1 | 张三      |   6 | M   | 2023-10-12 10:10:10 |
|  2 | 李四      |   7 | M   | 2023-10-12 11:10:10 |
|  3 | 王麻子    |   8 | F   | 2023-10-12 12:10:10 |
+----+-----------+-----+-----+---------------------+
3 rows in set (0.01 sec)

```



#### 方法二：创建容器时指定系统字符变量

在使用`run`穿件容器服务是，通过`-e LANG=C.utf8`指定系统变量，**此种方法在退出容器重新进入一直生效**

```shell
docker run -itd -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e LANG=C.utf8 --name mysql1 mysql
```

```shell
bash-4.4# locale
LANG=C.utf8
LC_CTYPE="C.utf8"
LC_NUMERIC="C.utf8"
LC_TIME="C.utf8"
LC_COLLATE="C.utf8"
LC_MONETARY="C.utf8"
LC_MESSAGES="C.utf8"
LC_PAPER="C.utf8"
LC_NAME="C.utf8"
LC_ADDRESS="C.utf8"
LC_TELEPHONE="C.utf8"
LC_MEASUREMENT="C.utf8"
LC_IDENTIFICATION="C.utf8"
LC_ALL=
```

