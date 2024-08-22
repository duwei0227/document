---
layout: post
title: 如何在Spring Bean创建销毁前执行自定义行为
categories: [Spring]
permalink: spring/bean.html
date: 2024-05-08 14:03:26
tags: Bean
---
`Bean`创建后和`Bean`销毁前执行自定义行为的三种方式。


### 一、实现 InitializingBean 和 DisposableBean 接口

`InitializingBean`在`Bean`创建完成后调用；`DisposableBean`在`Bean`销毁前调用

```java
public class Hello implements InitializingBean, DisposableBean {

    @Override
    public void destroy() throws Exception {
        System.out.println("destroy bean");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("bean init");
    }
}
```



### 二、使用 JSR-250 的 @PostConstruct 和 @PreDestroy 注解

`PostConstruct`在`Bean`创建完成后调用；`PreDestroy`在`Bean`销毁前调用

```java
@PostConstruct
public void init() {
    System.out.println("====== bean init ======");
}

@PreDestroy
public void beforeDestroy() {

}
```



### 三、在`<bean/>` 或 @Bean 里配置初始化和销毁方法

```java
@Bean(name = "hello", initMethod = "init", destroyMethod = "destroy")
```

