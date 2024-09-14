---
layout: post
title: 使用Spring Boot Admin管理和监控应用
categories: [Java]
tags: 监控
permalink: java/spring_boot_admin.html
---

`Spring Boot Admin`是一个监控工具，目的在与提供一个易于访问的方式可视化`Spring Boot Actuator`，它由两个主要部分组成：

* `server端：`提供用户界面来显示 `Spring Boot Actuators` 并与之交互
* `client端：`向`server端`注册并采集`Spring Boot Actuators`端点数据



### 一、Server

#### 1、依赖

```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>3.3.3</version>
</dependency>
```



#### 2、属性

| Property name                               | Description                            | Default value |
| ------------------------------------------- | -------------------------------------- | ------------- |
| `spring.boot.admin.server.enabled`          | 是否启用`Admin Server`                 | `true`        |
| `spring.boot.admin.context-path`            | 访问`Admin Server`路径，例如：`/admin` |               |
| `spring.boot.admin.monitor.status-interval` | 检查`Client`实例状态间隔（ms）         | 10,000ms      |



完整属性配置参考：[http://docs.spring-boot-admin.com/current/server.html](http://docs.spring-boot-admin.com/current/server.html)



启动后直接访问`Server`的地址，例如： `http://localhost:8080`,此处端口需要替换为实际端口，如果有指定 `context-path`，也需要在路径中加上。



### 二、Client

用于采集`Actuator Endpoints`数据并上报到`Server端`

#### 1、依赖

```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>3.3.3</version>
</dependency>
```



#### 2、属性

| 属性名称                                                     | 描述                                                         | 默认值                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------- |
| `spring.boot.admin.client.enabled`                           | 启用`Admin Client`                                           | `true`                       |
| `spring.boot.admin.client.url`                               | `Admin Server`的`URL`地址，存在多个的时候用逗号分割，例如：`http://localhost:8080` |                              |
| `spring.boot.admin.client.instance.name`                     | 注册示例名称                                                 | `${spring.application.name}` |
| `spring.boot.admin.client.username spring.boot.admin.client.password` | 用户名和密码（启用安全验证的情况）                           |                              |
| `spring.boot.admin.client.period`                            | 向`Server`发送注册信息的时间间隔(ms)                         | `10,000`                     |
| `spring.boot.admin.client.connect-timeout`                   | 连接超时时间(ms)                                             | `5,000`                      |
| `spring.boot.admin.client.read-timeout`                      | 读超时时间(ms)                                               | `5,000`                      |
| `spring.boot.admin.client.instance.metadata.*`               | `metadata`元信息                                             |                              |
| `spring.boot.admin.client.instance.metadata.tags.*`          | `Tag`信息，也会在`metadata`下显示                            |                              |



完整属性配置参考：[http://docs.spring-boot-admin.com/current/client.html](http://docs.spring-boot-admin.com/current/client.html)



