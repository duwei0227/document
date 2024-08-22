---
layout: post
title: Java普通类操作Spring Bean
categories: [Java]
permalink: spring/operation_bean.html
---

基于spring boot 1.5.6版本  
synchronized 防止实例同时访问修改

```Java
public final class SpringBeanTool implements ApplicationContextAware {
    private static ApplicationContext applicationContext = null;

    @Override
    public synchronized void setApplicationContext(ApplicationContext applicationContext) {
        if (SpringBeanTool.applicationContext == null) {
            SpringBeanTool.applicationContext = applicationContext;
        }
    }

    private static synchronized ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    public static Object getBean(Class clazz) {
        return getApplicationContext().getBean(clazz);
    }
}
```