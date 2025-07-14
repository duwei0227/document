---
layout: post
title: 使用Resilience4j CircuitBreaker实现服务调用熔断
categories: [Resilience4j]
permalink: resilience4j/circuitbreaker.html

---



断路器`CircitBreaker`是 Resilience4j 的核心机制，通过**有限状态机**和**环形位缓冲区（Ring Bit Buffer）** 实现故障检测与熔断

`CircitBreaker`采用装饰器模式对原有请求进行装饰，同时会增加方法执行时间的计算，方法执行结束后发生状态的流转。



## 一、基础概念

### 滑动窗口

滑动窗口可以类比为一个环形的圆圈，这个圆圈存储的元素上限就是`slidingWindowSize`。对于时间计数而言，这个圆圈代表的就是`slidingWindowSize`秒内接受处理的请求，对这些请求进行聚合计算，超过滑动窗口的调用会被丢弃；对于次数计数而言，这个圆圈代表的就是最近`slidingWindowSize`次调用，对最近的这些请求进行聚合计算。



### 状态流转

`Resilience4j`支持的状态包含三个普通状态：`CLOSED`、`OPEN`和`HALF_OPEN`以及三个特殊状态：`METRICS_ONLY`、`DISABLED`和`FORCED_OPEN`；熔断主要使用前三个状态。



![img](https://raw.githubusercontent.com/duwei0227/picbed/main/39cdd54-state_machine.jpg)



`CLOSED --> OPEN`状态变更：

前提：调用次数需要达到最小调用次数`minimumNumberOfCalls`要求

* 失败调用（默认所有异常都算作失败）比例超过阈值`ailureRateThreshold`
* 慢调用比例超过阈值`slowCallRateThreshold`



`OPEN --> HALF_OPEN`状态变更：

状态处于`OPEN`时，所有的请求会字节被熔断并抛出 `CallNotPermittedException` 异常。

前提：等待时间已经达到`OPEN`状态最大等待时间`waitDurationInOpenState`，熔断器尝试开始接受请求

* 启用自动状态转换配置`automaticTransitionFromOpenToHalfOpenEnabled`，会由监控线程将状态变更为`HALF_OPEN`接受请求
* 超过最大等待时间后，在尝试获取凭证时，会将状态变更为`HALF_OPEN`



`HALF_OPEN --> OPEN`状态变更：

* 允许执行请求的授权凭证`permittedNumberOfCallsInHalfOpenState`聚合计算后，失败比例或者慢调用还是超过阈值
* 设置`HALF_OPEN`状态最大等待时间`maxWaitDurationInHalfOpenState`非0时，超过等待时间，请请求记录无法计算阈值比较时，会将状态变更为`OPEN`



`HALF_OPEN --> CLOSED`状态变更：

* 失败比例和慢调用比例低于失败阈值和慢调用阈值



## 二、配置属性

### 1、滑动窗口定义

| 配置属性               | 默认值        | 描述                                                         |
| ---------------------- | ------------- | ------------------------------------------------------------ |
| `slidingWindowType`    | `count_based` | 滑动窗口类型，支持`count_based`和`time_based`，如果类型为`count_based`，最近的`slidingWindowSize`会记录和聚合，如果类型为`time_based`，最近的`slidingWindowSize`秒的请求会记录和聚合 |
| `slidingWindowSize`    | 100           | 滑动窗口的大小，当状态为`CLOSED`时记录请求结果               |
| `minimumNumberOfCalls` | 100           | 每个窗口周期最小调用次数，达到最小调用次数时才会用于失败率和慢调用。例如，配置值为10时，调用次数必须到达10次，才会进行结果计算，如果在窗口周期内只调用了9次，即使全部失败也不会触发熔断器计算 |



### 2、失败、超时阈值

| 配置属性                    | 默认值 | 描述                                                         |
| --------------------------- | ------ | ------------------------------------------------------------ |
| `failureRateThreshold`      | 50     | 失败阈值，百分比；如果失败比例大于等与阈值时，状态由`CLOSED`变为`OPEN` |
| `slowCallRateThreshold`     | 100    | 慢调用阈值，百分比；如果调用执行事件大于 `slowCallDurationThreshold`时认为是慢调用 |
| `slowCallDurationThreshold` | 60s    | 慢调用阈值，超过当前值考虑为慢调用                           |



### 3、状态轮转

| 配置属性                                       | 默认值  | 描述                                                         |
| ---------------------------------------------- | ------- | ------------------------------------------------------------ |
| `permittedNumberOfCallsInHalfOpenState`        | 10      | 状态为`HALF_OPEN`时允许通过的调用凭证数                      |
| `maxWaitDurationInHalfOpenState`               | 0[ms]   | `HALF_OPEN`状态保持的最大时间，超过该时间状态会变为`OPEN`；如果配置为0时会等待所有的凭证完成请求 |
| `automaticTransitionFromOpenToHalfOpenEnabled` | `false` | `true`:状态会自动由`OPEN`变为`HALF_OPEN`,不需要由请求触发，同时会创建一个线程监控实例是否已经达到`waitDurationInOpenState`时间<br/>`false`:状态不会自动发生转换，需要有请求发生触发状态变更为`HALF_OPEN`，同时也不会启动一个监控线程检查 |
| `waitDurationInOpenState`                      | 60s     | `OPEN`状态时，状态变为`HALF_OPEN`的最大等待时间，启用状态自动变化时才生效 |
| `enableExponentialBackoff`                     |         | 采用指数增长计算`OPEN`状态等待时间<br/>`wait_time = initial_interval * (multiplier ^ (retry_attempt - 1))` |
| `exponentialBackoffMultiplier`                 |         | 指数增长因子                                                 |
| `exponentialMaxWaitDurationInOpenState`        |         | 指数增长方式允许的最大值                                     |
| `enableRandomizedWait`                         |         | 是否采用随即时间等待，随机值：当前值 * 随机因子              |
| `randomizedWaitFactor`                         |         | 随机因子                                                     |



### 4、失败异常

| 配置属性                    | 默认值                               | 描述                                                         |
| --------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| `recordExceptions`          | `null`默认记录所有异常               | 需要作为失败记录的异常列表，和列表匹配或者时继承列表中的异常都会算作失败；如果指定了异常列表，默认其他异常都作为成功，除非明确在`ignoreException`列出 |
| `ignoreExceptions`          | `null`                               | 一个异常列表，即不作为成功也不作为失败，不会导致成功和失败数目增加 |
| `recordFailurePredicate`    | `throwable -> true` 默认记录所有异常 | 一个自定义的`Predicate`，用于判断异常是否作为失败异常记录    |
| `ignoreExceptionPredicate`  | `throwable -> false`                 | 一个自定义的`Predicate`，用于判断异常是否需要被忽略          |
| `writableStackTraceEnabled` | `true`                               | 是否打印详细日志堆栈，此配置只针对断路器处于`OPEN`状态时熔断请求时抛出异常`CallNotPermittedException` |



### 5、检测指标

* `resilience4j.circuitbreaker.metrics.enabled = true`
* `resilience4j.circuitbreaker.configs.defaults.register-health-indicator = true`
* `management.health.circuitbreakers.enabled = true`



## 三、使用示例

### 1、基础置

#### 1.1 `pom`依赖

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.2</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



#### 1.2 yaml配置

* `resilience4j.circuitbreaker.configs`：定义`CircuitBreaker`配置信息
* `resilience4j.circuitbreaker.instances`：定义`CircuitBreaker`实例信息
* `resilience4j.circuitbreaker.metrics`：定义`CircuitBreaker`指标信息

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'  # 暴露所有端口，可以根据需要暴露
  endpoint:
    health:
      show-details: always

  health:
    circuitbreakers:
      enabled: true  # 将 circuitbreaker 注册到 /actuator/health  

resilience4j:
  circuitbreaker:
    configs:	# 定义配置信息，可以在instance配置或者代码中引用
      default:   # 命名为 default 可以作为全局默认配置
        register-health-indicator: true
        sliding-window-size: 10  # 基于时间时，单位为秒，统计数据会随着时间窗口滚动丢弃超过10秒的数据
        sliding-window-type: time_based
        minimum-number-of-calls: 5
        slow-call-duration-threshold: 1s
        slow-call-rate-threshold: 50
        permitted-number-of-calls-in-half-open-state: 3  # 半打开状态时，也会计算比例
        automatic-transition-from-open-to-half-open-enabled: true   # 为true时，等待一定时间后自动变更状态为 half_open
        wait-duration-in-open-state: 15s
        enable-exponential-backoff: false # 是否开启指数递增失败等待
        exponential-backoff-multiplier: 2   # 乘数，当失败率超过阈值时，等待时间会乘以这个乘数，直到达到最大等待时间
        exponential-max-wait-duration-in-open-state: 5m
        enable-randomized-wait: false
        randomized-wait-factor: 0.5
        max-wait-duration-in-half-open-state: 0ms # 0ms 表示一直等待，直到所有的 permitted 调用完成，非0时达到最大等待时间后，会将状态变更为 open
        failure-rate-threshold: 50
        event-consumer-buffer-size: 10
#        writable-stack-trace-enabled: true   # 是否记录 CallNotPermittedException 异常完整堆栈信息

        record-exceptions:
          - org.springframework.web.client.HttpServerErrorException
          - java.util.concurrent.TimeoutException
          - java.io.IOException
        ignore-exceptions:
          - cn.probiecoder.spring_boot_resilience4j_demo.exception.BusinessException

      shared:
        sliding-window-type: count_based
        sliding-window-size: 8
        minimum-number-of-calls: 4
        automatic-transition-from-open-to-half-open-enabled: true
        permitted-number-of-calls-in-half-open-state: 2
        wait-duration-in-open-state: 3s
        failure-rate-threshold: 50
        event-consumer-buffer-size: 10
        ignore-exceptions:
          - cn.probiecoder.spring_boot_resilience4j_demo.exception.BusinessException
    instances:  # 定义 CircuitBreaker 实例
      backendA:
        base-config: default # 引用config配置
      backendB:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 10
        permitted-number-of-calls-in-half-open-state: 3
        wait-duration-in-open-state:
          seconds: 5
        failure-rate-threshold: 50
        event-consumer-buffer-size: 10
        record-failure-predicate: cn.probiecoder.spring_boot_resilience4j_demo.exception.RecordFailurePredicate
    metrics:
      enabled: true  # 启用指标数据监控
```



### 2、使用注解

#### 2.1 使用固定名称并指定fallback

```java
private static final String BACKEND_A = "backendA";

@CircuitBreaker(name = BACKEND_A, fallbackMethod = "fallback")
```

#### 2.2 使用方法名作为CircuitBreaker名称

```java
@CircuitBreaker(name = "#root.methodName")
```

#### 2.3 使用Spel表达时动态获取参数作为CircuitBreaker名称

```java
@CircuitBreaker(name = "#p0")

@CircuitBreaker(name = "#root.args[0]")

@CircuitBreaker(name = "#a0")
```



### 3、代码自定义装饰

```java
private final CircuitBreakerRegistry circuitBreakerRegistry;  // 使用全局的registry

public BackendAServiceImpl(CircuitBreakerRegistry circuitBreakerRegistry) {
    this.circuitBreakerRegistry = circuitBreakerRegistry;
}

public String customConfigFailure() {
    io.github.resilience4j.circuitbreaker.CircuitBreaker circuitBreaker = circuitBreakerRegistry.circuitBreaker("customConfig", "shared");


    Supplier<String> supplier = io.github.resilience4j.circuitbreaker.CircuitBreaker.decorateSupplier(circuitBreaker, () ->{
        throw new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "This is a remote exception");
    });

    // 此处使用 Try 需要引入 vavr 包
    return Try.ofSupplier(supplier).recover(th -> "Recovered: " + th.toString()).get();

}

```



## 四、附录

[https://github.com/resilience4j/resilience4j-spring-boot3-demo/tree/master](https://github.com/resilience4j/resilience4j-spring-boot3-demo/tree/master)

[https://resilience4j.readme.io/docs/circuitbreaker](https://resilience4j.readme.io/docs/circuitbreaker)

