---
layout: post
title: Spring 通过构造器自动注入所有子类
categories: [Spring]
permalink: spring/construct_autowired_subclass.html
---





**目的：**通过反射的方式动态调用子类方法



代码示例：

```java
@Component
public class RequestContent {
    private final Map<Class<? extends RequestHandler>, RequestHandler> handlerMap = new ConcurrentHashMap();
    
    @Autowired
    public RequestContext(Map<String, RequestHandler> handlers) {
        handler.forEach((k, v) -> this.handlerMap.put(v.getClass(), v));
    }
}
```

