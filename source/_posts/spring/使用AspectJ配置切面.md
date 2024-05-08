---
layout: post
title: 使用AspectJ配置切面
categories: [Spring]
description: 使用AspectJ配置切面
permalink: spring/aspectj.html
date: 2024-05-08 14:02:42
tags: AspectJ
---
基于 `@AspectJ`配置`AOP`,使用面向切面能力。


<!--more-->


####  声明切入点 `@Pointcut`

常用的切入点标识符,切入点可以参考对照方法的签名类记忆 **【修饰符(publci、private) 返回类型 包路径.方法名(参数列) 异常】**

其中方法修饰符，异常为可选项，可以不配置，其他为必须项，各部分都可以使用 ***** 表示任意类型

| 标识符      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| `execution` | 最常用的一个 PCD，用来匹配特定方法的执行                     |
| `within`    | 匹配特定范围内的类型，可以用通配符来匹配某个 Java 包内的所有类 |
| `this`      | Spring AOP 代理对象这个 Bean 本身要匹配某个给定的类型        |
| `target`    | 目标对象要匹配某个给定的类型，比 this 更常用一些             |
| `args`      | 传入的方法参数要匹配某个给定的类型，它也可以用于绑定请求参数 |
| `bean`      | Spring AOP 特有的一个 PCD，匹配 Bean 的 ID 或名称，可以用通配符 |



`execution`使用详细介绍（`[]`代表可选项，`<>`代表必选项）：

> execution([修饰符] <返回类型> [全限定类名.]<方法>(<参数>) [异常])

* 每个部分都可以使用 ***** 通配符
* 类名中使用 **.***表示包中的所有类，**..***表示当前包与子包中的所有类
* 参数主要分为以下几种情况：
  * `()`表示方法无参数
  * `(..)`表示有任意个参数
  * `(*)`表示有一个任意类型的参数
  * `(String)`表示有一个`String`类型的参数
  * `(String,String)`表示有两个`String`类型的参数

**切入点表达式支持与、或、非运算，运算符分别为 &&、||和 !**



#### 声明通知

##### 1、前置通知 `@Before`

```java
@Before(value = "execution(* cn.probiecoder.fund.aop.AspectJController.aspectj(..))")
public void before() {
    System.out.println("before....");
}
```



##### 2、后置通知 

* 拦截正常返回 `@AfterReturning`

  ```java
  @AfterReturning(value = "pointcut() && args()")
  public void afterReturning() {
      System.out.println("after method execute success");
  }
  ```

  

* 拦截一场返回 `@AfterThrowing`

  ```java
  @AfterThrowing(value = "pointcut() && args(throwException)")
  public void afterThrowing(boolean throwException) {
      System.out.println("请求参数："+throwException);
      System.out.println("after method execute throw exception");
  }
  ```

  

* 无论成功与否都执行 `@After`

  ```java
  @After(value = "execution(* cn.probiecoder.fund.aop..*.*(..))")
  public void after() {
      System.out.println("after method execute no matter success or error");
  }
  ```

  * 

##### 3、环绕通知

环绕通知不仅可以在方法执行前后加入自己的逻辑，甚至可以完全替换方法本身的逻辑，或者替换调用参数，可以通过`@Around`注解来声明环绕通知，这个注解**要求方法的第一个参数必须是`ProceedingJoinPoint`，方法的返回类型是被拦截方法的返回类型，或者直接用`Object`类型**。

```java
 @Around(value = "pointcut() && args(throwException)")
public Object around(ProceedingJoinPoint pjp, boolean throwException) {
    try {
        System.out.println("around before execute");
        System.out.println("请求参数：" + throwException);
        Object obj = pjp.proceed();
        System.out.println("around after execute");
        return obj;
    } catch (Throwable e) {
        throw new RuntimeException(e);
    }
}
```



##### 5、声明切入点

```java
@Pointcut("execution(* cn.probiecoder.fund.aop.AspectJController.aspectj(..))")
public void pointcut() {

}
```



#### 方法参数拦截

##### 1、args方法参数声明

格式：`args(参数变量名...)`

```java
args(throwException)
```



#### 注意事项

1、使用`AspectJ`需要在`pom`文件中引入相关依赖

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.21</version>
</dependency>
```

2、在启动类或者配置类上添加注解`@EnableAspectJAutoProxy`启用`AOP`切面编程支持

3、配置类上的`@Aspectj`仅仅是一个标识，不会被`Spring`识别为`Bean`进行注入，`AOP`不生效时，需要检查是否有添加`@Component`等注解声明类为`Bean`



#### 请求示例

```java
配置类：
package cn.probiecoder.fund.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class HelloAspectConfig {
    @Pointcut("execution(* cn.probiecoder.fund.aop.AspectJController.aspectj(..))")
    public void pointcut() {

    }

    @Before(value = "execution(* cn.probiecoder.fund.aop.AspectJController.aspectj(..))")
    public void before() {
        System.out.println("before....");
    }

    @AfterReturning(value = "pointcut() && args()")
    public void afterReturning() {
        System.out.println("after method execute success");
    }

    @AfterThrowing(value = "pointcut() && args(throwException)")
    public void afterThrowing(boolean throwException) {
        System.out.println("请求参数："+throwException);
        System.out.println("after method execute throw exception");
    }

    @After(value = "execution(* cn.probiecoder.fund.aop..*.*(..))")
    public void after() {
        System.out.println("after method execute no matter success or error");
    }

    @Around(value = "pointcut() && args(throwException)")
    public Object around(ProceedingJoinPoint pjp, boolean throwException) {
        try {
            System.out.println("around before execute");
            System.out.println("请求参数：" + throwException);
            Object obj = pjp.proceed();
            System.out.println("around after execute");
            return obj;
        } catch (Throwable e) {
            throw new RuntimeException(e);
        }
    }
}


拦截类：
@RestController
@RequestMapping("/aspectj")
public class AspectJController {
    @GetMapping
    public String aspectj(@RequestParam(value = "throwException", required = false) boolean throwException) {
        if (throwException) {
            throw new RuntimeException("抛出异常");
        }
        return "aspectj";
    }
}
