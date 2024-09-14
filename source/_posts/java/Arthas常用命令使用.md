---
layout: post
title: Arthas常用命令使用
categories: [Java]
---

运行 `arthas`：`java -jar arthas-boot.jar`



### 一、dashboard

通过`dashboard`可以用于监控线程状态，`CPU`使用情况以及内存使用(包含`GC`)情况。

Usage：

```shell
dashboard [-h] [-i <value>] [-n <value>] 
```

参数解释：

| 参数名称 | 参数说明                                 |
| -------: | :--------------------------------------- |
|     `-i` | 刷新实时数据的时间间隔 (ms)，默认 5000ms |
|     `-n` | 刷新实时数据的次数                       |
|     `-h` | 查看帮助                                 |



示例：

```shell
-- 每隔2秒刷新数据，总共刷新3次
dashboard -i 2000 -n 3
```



[https://arthas.aliyun.com/doc/dashboard.html#%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E](https://arthas.aliyun.com/doc/dashboard.html#%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E)



### 二、trace

`trace`主要用于分析接口各个方法执行用时，用于找出接口方法执行瓶颈。

Usage：

```shell
trace [-h] [-n <value>] [-m <value>] [-E] [--skipJDKMethod <value>] class-pattern method-pattern 
```



## 参数说明

|            参数名称 | 参数说明                                                |
| ------------------: | :----------------------------------------------------------- |
|     *class-pattern* | 类名表达式匹配，支持模糊匹配，通配符使用`*` | 
|    *method-pattern* | 方法名表达式匹配，支持模糊匹配，通配符使用`*` |
|                `-E` | 开启正则表达式匹配，默认为通配符匹配                         |
|                `-n` | 命令执行次数，默认值为 100。同一个方法最大跟踪次数        | 
|             `#cost` | 方法执行耗时                      |
|                `-m` | 指定 Class 最大匹配数量，默认值为 50。长格式为`[maxMatch <arg>]`。 |
|   `--skipJDKMethod` | 是否包含`jdk`方法，默认为 `true`        |



**示例：**

#### 1、使用默认配置

```shell
[arthas@12808]$ trace cn.probiecoder.adminclient.TestController hello
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 128 ms, listenerId: 1
`---ts=2024-09-13 13:22:48.208;thread_name=http-nio-8080-exec-1;id=50;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@1a1830fd
    `---[3316.403067ms] cn.probiecoder.adminclient.TestController:hello()
        +---[0.00% 0.019864ms ] kotlin.jvm.internal.Intrinsics:checkNotNullParameter()
        +---[6.43% 213.29613ms ] cn.probiecoder.adminclient.TestController:sleep_213_ms() #16
        +---[9.48% 314.234694ms ] cn.probiecoder.adminclient.TestController:sleep_314_ms() #17
        `---[14.33% 475.184939ms ] cn.probiecoder.adminclient.TestController:recall_5_times() #18
```



#### 2、根据耗时过滤

```kotlin
for (i in 1..5) {
    sleep_by_params(Random.nextLong(10, 200))
}

fun sleep_by_params(time: Long) {
    sleep(time)
}
```



```shell
[arthas@14400]$ trace cn.probiecoder.adminclient.TestController sleep_by_params '#cost > 100'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 70 ms, listenerId: 2
`---ts=2024-09-13 13:29:24.927;thread_name=http-nio-8080-exec-2;id=51;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@941d403
    `---[185.372153ms] cn.probiecoder.adminclient.TestController:sleep_by_params()

`---ts=2024-09-13 13:29:25.165;thread_name=http-nio-8080-exec-2;id=51;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@941d403
    `---[189.325869ms] cn.probiecoder.adminclient.TestController:sleep_by_params()

```



#### 3、正则模糊匹配包下所有类

```shell
[arthas@14400]$ trace -E cn.probiecoder..* hello
或
[arthas@14400]$ trace cn.probiecoder.* hello
Press Q or Ctrl+C to abort.
Affect(class count: 6 , method count: 1) cost in 87 ms, listenerId: 4
`---ts=2024-09-13 13:31:18.561;thread_name=http-nio-8080-exec-5;id=54;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@941d403
    `---[3914.62719ms] cn.probiecoder.adminclient.TestController:hello()
        +---[0.00% 0.035068ms ] kotlin.jvm.internal.Intrinsics:checkNotNullParameter()
        +---[5.45% 213.1525ms ] cn.probiecoder.adminclient.TestController:sleep_213_ms() #16
        +---[8.03% 314.217455ms ] cn.probiecoder.adminclient.TestController:sleep_314_ms() #17
        +---[9.49% 371.404243ms ] cn.probiecoder.adminclient.TestController:recall_5_times() #18
        +---[0.00% min=0.023609ms,max=0.027875ms,total=0.125867ms,count=5] kotlin.random.Random$Default:nextLong() #21
        `---[17.94% min=95.302312ms,max=172.21433ms,total=702.456271ms,count=5] cn.probiecoder.adminclient.TestController:sleep_by_params() #21

```



#### 4、模糊匹配类下边所有方法

```shell
[arthas@14400]$ trace cn.probiecoder.adminclient.TestController *
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 6) cost in 69 ms, listenerId: 6
`---ts=2024-09-13 13:34:45.418;thread_name=http-nio-8080-exec-9;id=58;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@941d403
    `---[3973.56132ms] cn.probiecoder.adminclient.TestController:hello()
        +---[0.00% 0.02265ms ] kotlin.jvm.internal.Intrinsics:checkNotNullParameter()
        +---[5.37% 213.475355ms ] cn.probiecoder.adminclient.TestController:sleep_213_ms() #16
        |   `---[99.96% 213.383829ms ] cn.probiecoder.adminclient.TestController:sleep_213_ms()
        +---[7.91% 314.269288ms ] cn.probiecoder.adminclient.TestController:sleep_314_ms() #17
        |   `---[99.98% 314.192024ms ] cn.probiecoder.adminclient.TestController:sleep_314_ms()
        +---[12.48% 495.794555ms ] cn.probiecoder.adminclient.TestController:recall_5_times() #18
        |   `---[99.94% 495.50564ms ] cn.probiecoder.adminclient.TestController:recall_5_times()

```



#### 5、指定方法匹配次数

```shell
[arthas@14400]$ trace cn.probiecoder.adminclient.TestController sleep_by_params -n 1
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 47 ms, listenerId: 9
`---ts=2024-09-13 13:37:18.101;thread_name=http-nio-8080-exec-3;id=52;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@941d403
    `---[186.244449ms] cn.probiecoder.adminclient.TestController:sleep_by_params()

Command execution times exceed limit: 1, so command will exit. You can set it with -n option.

```



#### 6、包含jdk函数

```shell
[arthas@17533]$ trace cn.probiecoder.adminclient.TestController hello --skipJDKMethod false
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 92 ms, listenerId: 2
`---ts=2024-09-13 13:40:03.255;thread_name=http-nio-8080-exec-2;id=51;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@3115e66
    `---[3689.6658ms] cn.probiecoder.adminclient.TestController:hello()
        +---[0.00% 0.027708ms ] kotlin.jvm.internal.Intrinsics:checkNotNullParameter()
        +---[0.00% 0.129751ms ] java.io.PrintStream:println() #15
        +---[62.67% 2312.325601ms ] java.lang.Thread:sleep() #16
        +---[5.78% 213.316132ms ] cn.probiecoder.adminclient.TestController:sleep_213_ms() #17
        +---[8.52% 314.354881ms ] cn.probiecoder.adminclient.TestController:sleep_314_ms() #18
        +---[11.24% 414.624544ms ] cn.probiecoder.adminclient.TestController:recall_5_times() #19
        +---[0.00% min=0.028151ms,max=0.038561ms,total=0.167493ms,count=5] kotlin.random.Random$Default:nextLong() #22
        `---[11.75% min=11.291751ms,max=176.242833ms,total=433.429189ms,count=5] cn.probiecoder.adminclient.TestController:sleep_by_params() #22
```



[https://arthas.aliyun.com/doc/trace.html](https://arthas.aliyun.com/doc/trace.html)



### 三、watch

使用`watch`观察指定函数的调用情况，能观察到的范围为：`返回值`、`抛出异常`、`入参`

Usage：

```shell
watch [-b] [-e] [-x <value>] [-f] [-h] [-n <value>] [-m <value>] [-E] [-M <value>] [-s] [-v] class-pattern method-pattern [express] [condition-express] 
```



| 参数名称            | 参数说明                                                     | 
| ------------------- | ------------------------------------------------------------ | 
| *class-pattern*     | 类名表达式匹配，支持模糊匹配，通配符使用`*`                  | 
| *method-pattern*    | 方法名表达式匹配，支持模糊匹配，通配符使用`*`                | 
| *express*           | 观察表达式，默认值：`{params, target, returnObj}`            |
| *condition-express* | 条件表达式                                                   | 
| `-b`                | 在**函数调用之前**观察 `--before`                            |
| `-e`                | 在**函数异常之后**观察 `--exception`                         |
| `-s`                | 在**函数返回之后**观察 `--success`                           |
| `-f`                | 在**函数结束之后**(正常返回和异常返回)观察 `--finish`        |
| `-E`                | 开启正则表达式匹配，默认为通配符匹配 `--regex`               | 
| `-x`                | 指定输出结果的属性遍历深度，默认为 1，最大值是 4，适合与对象多层嵌套情况 |



#### 1、观察函数调用返回时的参数、this 对象和返回值

```shell
arthas@17533]$ watch cn.probiecoder.adminclient.TestController hello
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 46 ms, listenerId: 5
method=cn.probiecoder.adminclient.TestController.hello location=AtExit
ts=2024-09-13 13:55:55.049; [cost=4074.771222ms] result=@ArrayList[
    @Object[][isEmpty=false;size=1],
    @TestController[cn.probiecoder.adminclient.TestController@1b95d54b],
    @String[hello_probie],
]

```

`location=AtExit`指明方法正常执行退出



#### 2、观察方法指定位置参数

参数位置通过 `[index]` 指定，可以通过逗号分割参数，如果使用`+`时，参数会被拼接

```shell
[arthas@17533]$ watch cn.probiecoder.adminclient.TestController hello '{params[0]}' -b
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 48 ms, listenerId: 9
method=cn.probiecoder.adminclient.TestController.hello location=AtEnter
ts=2024-09-13 14:00:02.530; [cost=0.022049ms] result=@ArrayList[
    @String[probie],
]

# 观察多个参数
[arthas@21542]$ watch cn.probiecoder.adminclient.TestController hello '{params[0],params[1]}' -b
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 62 ms, listenerId: 3
method=cn.probiecoder.adminclient.TestController.hello location=AtEnter
ts=2024-09-13 14:03:06.547; [cost=0.021551ms] result=@ArrayList[
    @String[probie],
    @Integer[3],
]

```



**表达式核心变量：**

|      变量名 | 变量解释                                                     |
| ----------: | :----------------------------------------------------------- |
|    `loader` | 本次调用类所在的 `ClassLoader`                               |
|     `clazz` | 本次调用类的 `Class` 引用                                    |
|    `method` | 本次调用方法反射引用                                         |
|    `target` | 本次调用类的实例                                             |
|    `params` | 本次调用参数列表，这是一个数组，如果方法是无参方法则为空数组 |
| `returnObj` | 本次调用返回的对象。当且仅当 `isReturn==true` 成立时候有效，表明方法调用是以正常返回的方式结束。如果当前方法无返回值 `void`，则值为 null |
|  `throwExp` | 本次调用抛出的异常。当且仅当 `isThrow==true` 成立时有效，表明方法调用是以抛出异常的方式结束。 |
|  `isBefore` | 辅助判断标记，当前的通知节点有可能是在方法一开始就通知，此时 `isBefore==true` 成立，同时 `isThrow==false` 和 `isReturn==false`，因为在方法刚开始时，还无法确定方法调用将会如何结束。 |
|   `isThrow` | 辅助判断标记，当前的方法调用以抛异常的形式结束。             |
|  `isReturn` | 辅助判断标记，当前的方法调用以正常返回的形式结束。           |





OGNL表达式官网：[https://commons.apache.org/dormant/commons-ognl/language-guide.html](https://commons.apache.org/dormant/commons-ognl/language-guide.html)

条件表达式核心变量：[https://arthas.aliyun.com/doc/advice-class.html](https://arthas.aliyun.com/doc/advice-class.html)

