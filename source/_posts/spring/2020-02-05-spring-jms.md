---
layout: post
title: Spring Boot JMS
categories: [JMS]
description: Spring Boot JMS
permalink: spring/jms.html
tags: jms
---

Spring Boot JMS

   使用spring-jms实现简单异步消息队列，用于异步通知以及异步事务处理。
项目：
* spring boot 2.2.2
* 基于内存模式的activiMQ消息队列
* 使用默认的Container

#### 一、pom依赖
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-broker</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

####  二、JMS config配置
进行文本的序列化(使用jackson)，支持对象参数  
自定义ContainerFactory(自定义配置以后，需要在Listener处指定containerFactory)
```java
/*
    自定义配置
     */
    @Bean
    public JmsListenerContainerFactory<?> myFactory(ConnectionFactory connectionFactory,
                                                    DefaultJmsListenerContainerFactoryConfigurer configurer) {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        configurer.configure(factory, connectionFactory);
        return factory;
    }

    @Bean // 序列化文本,可以支持对象参数
    public MessageConverter jacksonJmsMessageConverter() {
        MappingJackson2MessageConverter converter = new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");   // 不能为空,反序列化为object时需要
        return converter;
    }
```

#### 三、消息生产者
* destinationName 消息消费者名称  
* parameter 消费者所需参数，可以为对象
```java
@Autowired
private JmsTemplate jmsTemplate;

jmsTemplate.convertAndSend(destinationName, parameter);
```

#### 四、消息消费者
* destination 定义消费者名称，用户生产者调用；必须
* containerFactory 指定自定义Container; 可选
```java
@JmsListener(destination = "mailbox", containerFactory = "myFactory")
```

#### 五、其他
需要在启动类允许jms `@EnableJms`