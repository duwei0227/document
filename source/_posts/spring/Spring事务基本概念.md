---
layout: post
title: Spring事务基本概念
categories: [Spring]
permalink: spring/transaction.html
date: 2024-05-08 14:02:29
---

#### 事务传播性

| 传播性                      | 值   | 描述                                             |
| --------------------------- | ---- | ------------------------------------------------ |
| `PROPAGATION_REQUIRED`      | 0    | 当前有事务就用当前事务，没有事务就启动一个新事务 |
| `PROPAGATION_SUPPORTSP`     | 1    | 事务不是必须的，可以有事务，也可以没有           |
| `PROPAGATION_MANDATORY`     | 2    | 一定要存在一个事务，不然就报错                   |
| `PROPAGATION_REQUIRES_NEW`  | 3    | 新启动一个事务，如果当前存在一个事务就将其挂起   |
| `PROPAGATION_NOT_SUPPORTED` | 4    | 不支持事务，以非事务的方式运行                   |
| `PROPAGATION_NEVER`         | 5    | 不支持事务，如果当前存在一个事务则抛异常         |
| `PROPAGATION_NESTED`        | 6    | 如果当前存在一个事务，则在该事务内在启动一个事务 |



#### 隔离级别

| 隔离性                       | 描述     | 取值 | 脏读   | 不可重复读 | 幻读   |
| ---------------------------- | -------- | ---- | ------ | ---------- | ------ |
| `ISOLATION_READ_UNCOMMITTED` | 读未提交 | 1    | 存在   | 存在       | 存在   |
| `ISOLATION_READ_COMMITTED`   | 读已提交 | 2    | 不存在 | 存在       | 存在   |
| `ISOLATION_REPEATABLE_READ` | 可重复读 | 3    | 不存在 | 不存在 | 存在   |
| `ISOLATION_SERIALIZABLE`    | 串行读   | 4    | 不存在 | 不存在 | 不存在 |



#### 使用 @Transaction 注解声明事务

| 属性名                                 | 属性描述                   | 默认值                                        |
| -------------------------------------- | -------------------------- | --------------------------------------------- |
| `transactionManager`                   | 指定事务管理器             | 默认查找`transactionManager`的事务管理器      |
| `propagation`                          | 指定事务的传播性           | `Propagation.REQUIRED`                        |
| `isolation`                            | 指定事务的隔离级别         | `Isolation.DEFAULT`取决于数据库本身的事务级别 |
| `timeout`                              | 指定事务超时事件           | -1，由具体的底层实现来设置                    |
| `readonly`                             | 是否为只读事务             | `false`                                       |
| `rollbackFor/rollbackForClassName`     | 指定需要回滚事务的异常类型 | 无   |
| `noRollbackFor/noRollbackForClassName` | 指定无需回滚事务的异常类型 | 无   |



**注意事项：**`Spring`的声明式事务，其本质是对目标类和方法进行了`AOP`拦截，并在方法的执行前后增加了事务相关的操作，比如启动事务、提交事务和回滚事务。必须调用增强后的代理类中的方法，而非原本的对象，这样才能拥有事务。

