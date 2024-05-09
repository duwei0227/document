---
layout: post
title: Spring自定义事件
categories: [Spring]
description: Spring自定义事件
permalink: spring/event.html
date: 2024-05-08 14:02:54
tags: Event
---
ApplicationContext 提供了一套事件机制，在容器发生变动时我们可以通过 ApplicationEvent 的子类通知到 ApplicationListener 接口的实现类，做对应的处理。例如，ApplicationContext 在启动、停止、关闭和刷新 20 时，分别会发出 ContextStartedEvent、ContextStoppedEvent、ContextClosedEvent 和 ContextRefreshedEvent 事件，这些事件就让我们有机会感知当前容器的状态。


<!--more-->


如下两种方式可以进行自定义事件的发布和监听。



#### 方式一：实现接口发布、监听事件

* 事件监听 实现 `ApplicationListener` 接口

  ```java
  package cn.probiecoder.fund.event;
  
  import org.springframework.context.ApplicationListener;
  import org.springframework.stereotype.Component;
  
  @Component
  public class AgreeEventListener implements ApplicationListener<AgreeEvent> {
      @Override
      public void onApplicationEvent(AgreeEvent event) {
          System.out.println("我是监听器");
          System.out.println(event.getMessage());
      }
  }
  
  ```

  

* 事件发布 实现 `ApplicationEventPublisherAware` 接口

```java
package cn.probiecoder.fund.event;

import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/event")
public class EventController implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher publisher;
    @GetMapping
    public String publicEvent() {
        AgreeEvent agreeEvent = new AgreeEvent(this, "hello, event");
        publisher.publishEvent(agreeEvent);
        return "publicEvent";
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher = applicationEventPublisher;
    }
}

```

#### 方式二、使用注解
发布事件：
```java
@Autowired
private ApplicationEventPublisher publisher;
```

使用注解`@EventListener`监听事件：
```java
@EventListener
public void onApplicationEvent(AgreeEvent event) {
    System.out.println("我是监听器");
    System.out.println(event.getMessage());
}

```

**自定义事件：**
```java
package cn.probiecoder.fund.event;

import org.springframework.context.ApplicationEvent;

public class AgreeEvent extends ApplicationEvent {
    private String message;

    public AgreeEvent(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return this.message;
    }
}
```