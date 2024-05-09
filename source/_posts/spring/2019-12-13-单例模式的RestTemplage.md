---
layout: post
title: 单例模式的RestTemplate
categories: [Java]
description: 单例模式的RestTemplate
permalink: spring/singleton_resttemplate.html
tags: RestTemplate
---

在`Spring Boot` 升级到 `2.0` 以后，无法使用 `@Autowired` 直接注入 `RestTemplate` , 需要通过 `RestTemplateBuilder` 产生。基于此实现了一个单例模式的工具类，用于获取 `RestTemplate`，同时学习单例模式的写法。   

以下方法基于 `Spring Boot 2.1.5.RELEASE`测试通过

本文实现了三种方式：  
#### 1、线程安全的懒汉模式
由于方法锁的缘由，在并发访问的时候效率会降低
```java
public class RestTemplateUtil {
    private static RestTemplate restTemplate;

    private RestTemplateUtil() {

    }

    public static synchronized RestTemplate getRestTemplate() {
        if (restTemplate == null) {
            restTemplate = new RestTemplateBuilder().builder();
        }
        return restTemplate;
    }
}
```

#### 2、饿汉模式
在类加载的时候初始化，常驻内存
```java
public class RestTemplateUtil {
    private static final RestTemplate restTemplate = new RestTemplateBuilder().builder();

    private RestTemplateUtil() {

    }

    public static synchronized RestTemplate getRestTemplate() {
        return restTemplate;
    }
}
```

#### 3、枚举方式 
推荐方法，参考《Effective Java》
* 自有序列号  
* 保证只有一个实例  
* 线程安全
  
```java
public enum RestTemplateUtil {
    INSTANCE;

    public RestTemplate getRestTemplate() {
        return new RestTemplateBuilder().builder()
    }
}
```

#### 4、私有类模式
```java
public class RestTemplateUtil {
    private static class LazySingleton {
        private static final Singleton INSTANCE = new RestTemplateUtil();
    }

    public static RestTemplateUtil getRestTemplateUtil() {
        return LazySingleton.INSTANCE;
    }
}
```