---
layout: post
title: 获取Spring容器
categories: [Spring]
description: 获取Spring容器
permalink: spring/container.html
date: 2024-05-08 14:03:05
---
在普通`Bean`中获取`Spring`容器，可以通过实现 `Aware`接口或者直接注入，方式如下。

优先建议注入`ApplicationContext`，`ApplicationContext`扩展了`BeanFactory`功能



#### 方式一：实现 BeanFactoryAware 或 ApplicationContextAware 接口

```java
implements ApplicationContextAware
     
@Override
public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    this.ac = applicationContext;
}


implements BeanFactoryAware
@Override
public void setBeanFactory(BeanFactory beanFactory) throws BeansException {

}
```



此种方式适合于普通的`Java`类



#### 方式二：用 @Autowired 注解或构造器来注入 BeanFactory 或 ApplicationContext

此种方式适合于`Spring Bean`中注入



获取到`ApplicationContext`后可以用来获取`Bean`，并执行`Bean`的相关方法，获取环境资源文件配置









