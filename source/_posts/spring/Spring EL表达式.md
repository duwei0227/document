---
layout: post
title: Spring EL表达式
categories: [Spring]
permalink: spring/spring_el.html
---

#### 

### 一、Spring拦截器中获取自定义注解并计算el表达式

| 类名                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| `ExpressionParser`  | 表达式的解析器，主要用来生产表达式                           |
| `Expression`        | 表达式对象                                                   |
| `EvaluationContext` | 表达式运行的上下文。包含了表达式运行时的参数，以及需要执行方法的类 |



#### 1、自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CompletionInstance {
    String instanceId();
}
```



#### 2、注解使用SpEL

```java
@CompletionInstance(instanceId = "#dto.instanceId")
```

`dto`为一个对象

```java
@Data
public class Dto {
    private Long instanceId;
}
```



#### 3、定义Aspect拦截

```java
@Aspect
@Component
@Order
@Slf4j
public class CompletionInstanceAspect {
    @Around(value = "@annotation(completionInstance)")
    public Object doProcess(ProceedingJoinPoint pjp, CompletionInstance completionInstance) {
        // xxx
    }
}
```



* `@Order`指定`Aspect`拦截优先级
* `@annotation`括号中为方法定义的需要拦截的注解参数名



#### 4、解析SpEL获取实际值

```java
MethodSignature signature = (MethodSignature) pjp.getSignature();
// 获取方法参数
Object[] args = pjp.getArgs();

// 创建SpEL解析器
ExpressionParser parser = new SpelExpressionParser();

// 创建SpEL上下文
StandardEvaluationContext context = new StandardEvaluationContext();
// 此处设置SpEL表达式计算所需变量
String[] parameterNames = signature.getParameterNames();
for (int i = 0; i < parameterNames.length; i++) {
    context.setVariable(parameterNames[i], args[i]);
}

// 获取注解上的SpEL表达式
String expressionString = completionInstance.instanceId();

// 解析和计算SpEL表达式
Expression expression = parser.parseExpression(expressionString);
Long instanceId = expression.getValue(context, Long.class);
```



**可以将SpEL解析逻辑按需抽取**