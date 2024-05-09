---
layout: post
title: 使用Python操作MySql
categories: [Python]
description: 使用Python操作MySql
tags: Pymysql
permalink: python/pymysql.html
---

### 一、准备
* python `3.7.4 docker`
* mysql `Ver 8.0.18 for Linux on x86_64` docker
* PyMySQL `pip install PyMySQL`

### 二、注意事项
* mysql host需要使用docker容器内部分配的ip(使用 `ifconfig`查看) -- idea可以直接使用`localhost:映射端口`连接，有一定误导
* 事务不会自动提交，需要主动控制 `commit`或`rollback`
* `drop`操作需要数据表实际存在（`drop table if exists user`）
* `create table`中`int(10)`在创建后的数据库表中字段大小**10**已经丢失，在当前数据库版本中会有警告
* 实际值大小要和字段匹配
* 占位符使用`%s`,参数使用元组(`(p,)`)
* 连接关闭建议使用`with`上下文管理，保证资源释放

### 三、主要代码
#### 1、连接
```python
self.db = pymysql.connect(host=self.host,
                                      user=self.user,
                                      password=self.password,
                                      database=self.database,
                                      port=self.port,
                                      charset=self.charset,
                                      cursorclass=self.cursorclass)
self.cursor = self.db.cursor()
```
#### 2、执行`execute`
`execute` 返回命中记录数
通过`cursor.fetchone`、`fetchall`、`fetchmany(size)`获取实际数据
```python
select_sql = 'select id, name, cellphone, balance, create_time, status from user where id = %s'

data_id = 5  # 在insert完成结果中返回是最好的,待解决
cursor.execute(select_sql, (data_id, ))
result = cursor.fetchone()  # 返回为dict, 一条
# result = cursor.fetchall()  # 返回为dict，全部
# result = cursor.fetchmany(size=5)  # 返回为dict，指定大小
```

#### 3、事务
此处使用数据库`connect`操作事务
```python
self.connect.commit()
self.connect.rollback()
```
#### 4、连接关闭
先关闭`cursor`在关闭`connect`
```python
cursor.close()
connect.close()
```

参考：
[菜鸟教程](https://www.runoob.com/python3/python3-mysql.html)