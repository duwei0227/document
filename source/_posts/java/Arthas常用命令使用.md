## 一、启动Arthas

```java
java -jar arthas-boot.jar
```



## 二、表达式核心变量

匹配表达式和观察表达式核心判断变量

|    变量名 | 变量解释                                                     |
| --------: | :----------------------------------------------------------- |
|    loader | 本次调用类所在的 ClassLoader                                 |
|     clazz | 本次调用类的 Class 引用                                      |
|    method | 本次调用方法反射引用                                         |
|    target | 本次调用类的实例                                             |
|    params | 本次调用参数列表，这是一个数组，如果方法是无参方法则为空数组 |
| returnObj | 本次调用返回的对象。当且仅当 `isReturn==true` 成立时候有效，表明方法调用是以正常返回的方式结束。如果当前方法无返回值 `void`，则值为 null |
|  throwExp | 本次调用抛出的异常。当且仅当 `isThrow==true` 成立时有效，表明方法调用是以抛出异常的方式结束。 |
|  isBefore | 辅助判断标记，当前的通知节点有可能是在方法一开始就通知，此时 `isBefore==true` 成立，同时 `isThrow==false` 和 `isReturn==false`，因为在方法刚开始时，还无法确定方法调用将会如何结束。 |
|   isThrow | 辅助判断标记，当前的方法调用以抛异常的形式结束。             |
|  isReturn | 辅助判断标记，当前的方法调用以正常返回的形式结束。           |

官网文档：[https://arthas.aliyun.com/doc/advice-class.html](https://arthas.aliyun.com/doc/advice-class.html)



## 三、命令列表

| 命令          | 描述                                             |
| ------------- | ------------------------------------------------ |
| `dashboard`   | 当前系统的实时数据面板--直观查看CPU和内存使用    |
| `getstatic`   | 查看类的静态属性                                 |
| `heapdump`    | 打印堆信息，类似与 `jmap`                        |
| `jvm`         | 查看当前`JVM`的信息                              |
| `logger`      | 查看和修改`logger`                               |
| `memory`      | 查看`JVM`的内存信息                              |
| `thread`      | 查看当前`JVM`的线程堆栈信息                      |
| `sysenv`      | 查看`JVM`的环境变量--系统环境变量信息            |
| `sysprop`     | 查看和修改`JVM`的系统属性                        |
| `vmoption`    | 查看和修改 JVM 里诊断相关的 option               |
| `classloader` | 查看`classloader`信息                            |
| `jad`         | 反编译已加载类的源码                             |
| `sc`          | 查看`JVM`已加载的类信息                          |
| `sm`          | 查看`JVM`已加载类的方法信息                      |
| `stack`       | 输出当前方法被调用的调用路径                     |
| `trace`       | 方法内部调用路径，并输出方法路径上每个节点的耗时 |
| `watch`       | 方法执行数据观测                                 |



### 1、dashboard

**参数说明**

| 参数名称 | 参数说明                                 |
| -------: | :--------------------------------------- |
|     `-i` | 刷新实时数据的时间间隔 (ms)，默认 5000ms |
|     `-n` | 刷新实时数据的次数                       |



**使用示例**

每3秒刷新一次数据，总共刷新3次

```shell
dashbaord -i 3000 -n 3
```

![image-20250507083504830](https://raw.githubusercontent.com/duwei0227/picbed/main/image-20250507083504830.png)



**CPU数据说明**

- ID: Java 级别的线程 ID，注意这个 ID 不能跟 jstack 中的 nativeID 一一对应。
- NAME: 线程名
- GROUP: 线程组名
- PRIORITY: 线程优先级, 1~10 之间的数字，越大表示优先级越高
- STATE: 线程的状态
- CPU%: 线程的 cpu 使用率。比如采样间隔 1000ms，某个线程的增量 cpu 时间为 100ms，则 cpu 使用率=100/1000=10%
- DELTA_TIME: 上次采样之后线程运行增量 CPU 时间，数据格式为`秒`
- TIME: 线程运行总 CPU 时间，数据格式为`分:秒`
- INTERRUPTED: 线程当前的中断位状态
- DAEMON: 是否是 daemon 线程



### 2、getstatic

查看类的静态属性，支持查看`private`和`public`属性，官网推荐直接使用`ognl`

**语法**

```shell
getstatic [-c <value>] [-x <value>] [-E] class-pattern field-pattern [express] 
```

**参数说明**

| 参数名称        | 参数说明                                                     |
| --------------- | ------------------------------------------------------------ |
| `-c`            | 指定`classloader`哈希值，可以通过`sc -d`获取哈希值或 `classloader -t`获取 |
| `class-pattern` | 需要查看的类全路径                                           |
| `field-pattern` | 需要查看的属性,支持模糊匹配,如果添加选项`-E`时，会按照严格模式匹配 |



**使用示例**

```shell
// 获取属性
[arthas@14691]$ getstatic cn.probiecoder.arthas_demo.TestController map
field: map
@HashMap[
    @String[key1]:@String[value1],
    @String[key2]:@String[value2],
    @String[key3]:@String[value3],
]
  
// 查看类下边所有属性
[arthas@14691]$ getstatic -c 76ed5528 cn.probiecoder.arthas_demo.TestController *
field: MESSAGE
@String[Hello, Arthas!]
field: GETSTATIC_MESSAGE
@String[getStaticMessage]
field: map
@HashMap[
    @String[key1]:@String[value1],
    @String[key2]:@String[value2],
    @String[key3]:@String[value3],
]

// 模糊匹配属性    
[arthas@14691]$ getstatic -c 76ed5528 cn.probiecoder.arthas_demo.TestController *M*
field: MESSAGE
@String[Hello, Arthas!]
field: GETSTATIC_MESSAGE
@String[getStaticMessage]
       
```



### 3、heapdump

打印堆信息，类似于`jmap`

**语法**

```java
heapdump [-l] [file]
```

**参数说明**

| 参数名称 | 参数说明                                    |
| -------- | ------------------------------------------- |
| `-l`     | 只 `dump live` 对象，不添加时`dump`所有对象 |

**使用示例**

```shell
// dump所有对象
[arthas@14691]$ heapdump /home/duwei/arthas-output/heap.hprof
Dumping heap to /home/duwei/arthas-output/heap.hprof ...
Heap dump file created

// dump live 对象
[arthas@14691]$ heapdump -l /home/duwei/arthas-output/heap_live.hprof
Dumping heap to /home/duwei/arthas-output/heap_live.hprof ...
Heap dump file created
```



`dump`时需要确保路径已经创建，否则会报错：`heap dump error: No such file or directory`



### 4、jvm

查看当前 JVM 信息，会输出当前内存、线程、GC等信息

**语法**

```shell
jvm
```

**使用示例**

```shell
// 会输出以下类别信息，具体内容省略
[arthas@14691]$ jvm
 RUNTIME                                                                                           
 CLASS-LOADING                                                                                           
 COMPILATION                                                                                         
 GARBAGE-COLLECTORS                                                                        
 MEMORY-MANAGERS                                                                                  
 MEMORY                                                                                
 OPERATING-SYSTEM                                                                                    
 THREAD                                                                                         
 FILE-DESCRIPTOR                             
```



### logger

查看`logger` 信息，更新 `logger level`，临时调整日志记录级别用于问题定位会很有用



**语法**

```shell
logger [-c <value>] [-l <value>] [-n <value>] 
```



**参数说明**

| 参数名称 | 参数说明                                                    |
| -------- | ----------------------------------------------------------- |
| `-c`     | 指定`classloader`，可以通过`sc -d`获取`classloader`的哈希值 |
| `-l`     | 指定`level`级别                                             |
| `-n`     | 指定`logger`名字，这里可以为类路径                          |

**使用示例**

```shell
// 查看所有的logger
[arthas@44455]$ logger
 name             ROOT                                                                                  
 class            ch.qos.logback.classic.Logger                                                         
 classLoader      jdk.internal.loader.ClassLoaders$AppClassLoader@76ed5528                              
 classLoaderHash  76ed5528                                                                              
 level            INFO                                                                                  
 effectiveLevel   INFO  
 
// 更新指定类或包的日志级别
[arthas@44455]$ logger -n cn.probiecoder.arthas_demo.TestController -l debug
Update logger level success.
[arthas@44455]$ logger -n cn.probiecoder.arthas_demo -l debug
Update logger level success.

```



### 5、memory

查看`JVM`内存信息，如果只查看`heap`和`nonheap`使用情况，可以使用`dashboard`动态刷新监控

**语法**

```shell
memory
```



**使用示例**

可以使用`java.nio.ByteBuffer.allocateDirect(1024 * 1024 * 10); // 10MB`分配对外直接内存

```shell
[arthas@44455]$ memory 
Memory                                       used          total          max            usage          
heap                                         37M           76M            3918M          0.95%          
g1_eden_space                                10M           40M            -1             25.00%         
g1_old_gen                                   25M           34M            3918M          0.66%          
g1_survivor_space                            1M            2M             -1             65.71%         
nonheap                                      86M           88M            -1             97.96%         
codeheap_'non-nmethods'                      1M            2M             5M             33.16%         
metaspace                                    57M           58M            -1             98.68%         
codeheap_'profiled_nmethods'                 15M           15M            117M           13.29%         
compressed_class_space                       6M            7M             1024M          0.67%          
codeheap_'non-profiled_nmethods'             5M            5M             117M           4.28%          
mapped                                       0K            0K             -              0.00%          
direct                                       14M           14M            -              100.00%        
mapped - 'non-volatile memory'               0K            0K             -              0.00%  
```



### 6、thread

查看当前线程信息，查看线程的堆栈

**语法**

```shell
thread [--all] [-b] [-i <value>] [--state <value>] [-n <value>] [id]    
```



**参数说明**

| 参数名称  | 参数说明                                                     |
| --------- | ------------------------------------------------------------ |
| `--all`   | 列出所有线程                                                 |
| `-b`      | 找出当前阻塞其他线程的线程                                   |
| `-i`      | 指定采样时间间隔                                             |
| `--state` | 根据线程状态过滤，`NEW, RUNNABLE, TIMED_WAITING, WAITING, BLOCKED, TERMINATED` |
| `-n`      | 指定要查看的线程数量，按照`CPU`使用率降序排序                |
| `id`      | 线程id，查看指定线程的堆栈信息                               |

**使用示例**

```shell
// 死锁检测
[arthas@44455]$ thread -b
"Thread-4" Id=80 BLOCKED on java.lang.Object@12ea72d3 owned by "Thread-5" Id=81
    at app//cn.probiecoder.arthas_demo.TestController.lambda$0(TestController.java:89)
    -  blocked on java.lang.Object@12ea72d3
    -  locked java.lang.Object@16629d6a <---- but blocks 1 other threads!
    at app//cn.probiecoder.arthas_demo.TestController$$Lambda/0x00007fa97475dcd0.run(Unknown Source)
    at java.base@21.0.6/java.lang.Thread.runWith(Thread.java:1596)
    at java.base@21.0.6/java.lang.Thread.run(Thread.java:1583)


// 根据状态过滤
[arthas@44455]$ thread --state BLOCKED
Threads Total: 39, NEW: 0, RUNNABLE: 17, BLOCKED: 2, WAITING: 13, TIMED_WAITING: 7, TERMINATED: 0       
ID  NAME                      GROUP        PRIORITY STATE    %CPU    DELTA_TI TIME     INTERRU DAEMON   
80  Thread-4                  main         5        BLOCKED  0.02    0.000    0:0.002  false   true     
81  Thread-5                  main         5        BLOCKED  0.01    0.000    0:0.002  false   true     
  
// 查看最繁忙的3个线程，采样时间为3秒内
[arthas@44455]$ thread -i 3000 -n 3

```



### 7、sysenv

查看当前`JVM`的环境属性，操作系统环境变量信息

**语法**

```shell
sysenv [env-name]  
```



**参数说明**

| 参数名称   | 参数说明                                           |
| ---------- | -------------------------------------------------- |
| `env-name` | 可选，默认展示所有的环境属性，指定时输出指定属性值 |

**使用示例**

```shell
[arthas@44455]$ sysenv LANG
 KEY                  VALUE                                                                             
--------------------------------------------------------------------------------------------------------
 LANG                 en_US.UTF-8   
```



### 8、sysprop

查看或修改当前`JVM` 的系统属性

**语法**

```shell
sysprop [property-name] [property-value]  
```



**参数说明**

| 参数名称         | 参数说明                  |
| ---------------- | ------------------------- |
| `property-name`  | 属性名称,默认输出所有属性 |
| `property-value` | 属性修改后的值            |

**使用示例**

```shell
[arthas@44455]$ sysenv spring.profiles.active
 KEY                  VALUE                                                                             
--------------------------------------------------------------------------------------------------------
 spring.profiles.active  null            
 
[arthas@44455]$ sysprop spring.profiles.active dev
Successfully changed the system property.
 KEY                  VALUE                                                                             
--------------------------------------------------------------------------------------------------------
 spring.profiles.active  dev                                                                               

[arthas@44455]$ sysprop spring.profiles.active 
 KEY                  VALUE                                                                             
--------------------------------------------------------------------------------------------------------
 spring.profiles.active  dev                                                                               
```



### 9、vmoption

查看，更新 VM 诊断相关的参数，在更新`option`时，`option`需要支持运行时动态更新

**语法**

```shell
vmoption [name] [value]                                                                         
```



**参数说明**

| 参数名称 | 参数说明       |
| -------- | -------------- |
| `name`   | `option`的名称 |
| `value`  | 修改后的值     |

**使用示例**

```shell
// 查看指定option
[arthas@44455]$ vmoption HeapDumpBeforeFullGC
 KEY                       VALUE                     ORIGIN                    WRITEABLE                
--------------------------------------------------------------------------------------------------------
 HeapDumpBeforeFullGC      false                     DEFAULT                   true                     
// 修改指定option
[arthas@44455]$ vmoption HeapDumpBeforeFullGC true
Successfully updated the vm option.
 NAME                  BEFORE-VALUE  AFTER-VALUE                                                        
-------------------------------------------------                                                       
 HeapDumpBeforeFullGC  false         true                                                               
[arthas@44455]$ vmoption HeapDumpBeforeFullGC
 KEY                       VALUE                     ORIGIN                    WRITEABLE                
--------------------------------------------------------------------------------------------------------
 HeapDumpBeforeFullGC      true                      MANAGEMENT                true      
```



### 10、classloader

展示`JVM`中所有的`classloader`信息，可以加载指定类

**语法**

```shell
classloader -t
```



**参数说明**

| 参数名称 | 参数说明                  |
| -------- | ------------------------- |
| `-t`     | 查看`ClassLoader`的继承树 |

**使用示例**

```shell
[arthas@44455]$ classloader -t
+-BootstrapClassLoader                                                                                  
+-jdk.internal.loader.ClassLoaders$PlatformClassLoader@4ca1ec81                                         
  +-com.taobao.arthas.agent.ArthasClassloader@734cb0e4                                                  
  +-jdk.internal.loader.ClassLoaders$AppClassLoader@76ed5528                                            
    +-sun.reflect.misc.MethodUtil@1af4d51c   
```



### 11、jad

反编译指定已加载类的源码

**语法**

```shell
jad [-c <value>] [--lineNumber <value>] [--source-only] class-pattern [method-name] 
```



**参数说明**

| 参数名称        | 参数说明                                                     |
| --------------- | ------------------------------------------------------------ |
| `-c`            | 指定`classloader`的哈希值                                    |
| `--lineNumber`  | 反编译以后是否显示行号 `true` `false`                        |
| `--source-only` | 默认情况下，反编译结果里会带有`ClassLoader`信息，通过`--source-only`选项，可以只打印源代码 |
| `class-pattern` | 需要反编译的类路径                                           |
| `method-name`   | 方法名，支持指定方法反编译                                   |

**使用示例**

```shell
[arthas@44455]$ jad -c 76ed5528 --source-only --lineNumber false cn.probiecoder.arthas_demo.TestController  threadPoolBlock
@GetMapping(value={"/thread_pool_block"})
public String threadPoolBlock() {
    ExecutorService executor = Executors.newFixedThreadPool(1);
    executor.submit(() -> {
        try {
            Thread.sleep(10000L);
        }
        catch (InterruptedException e) {sc
            Thread.currentThread().interrupt();
        }
    });
    return "Thread pool block test started";
}

```



### 12、sc(search-class)

查看 JVM 已加载的类信息

**语法**

```shell
sc [-d] [-c <value>] [-f] class-pattern
```



**参数说明**

| 参数名称        | 参数说明                                                     |
| --------------- | ------------------------------------------------------------ |
| `-c`            | 指定`classloader`哈希值                                      |
| `-f`            | 输出当前类的成员变量信息（需要配合参数-d 一起使用）          |
| `-d`            | 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的 ClassLoader 等详细信息。<br/>如果一个类被多个 ClassLoader 所加载，则会出现多次 |
| `class-pattern` | 类路径                                                       |

**使用示例**

```shell
[arthas@73802]$ sc -d -f cn.probiecoder.arthas_demo.TestController 
 class-info        cn.probiecoder.arthas_demo.TestController                                            
 name              cn.probiecoder.arthas_demo.TestController                                            
 isInterface       false                                                                                
 isAnnotation      false                                                                                
 isEnum            false                                                                                
 isAnonymousClass  false     
 .....
```



### 13、sm

查看已加载类的方法信息

**语法**

```shell
sm [-c <value>][-d] class-pattern [method-pattern] 
```



**参数说明**

| 参数名称         | 参数说明                                                     |
| ---------------- | ------------------------------------------------------------ |
| `-c`             | 指定`classloader`哈希值                                      |
| `-d`             | 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的 ClassLoader 等详细信息。<br/>如果一个类被多个 ClassLoader 所加载，则会出现多次 |
| `class-pattern`  | 类路径                                                       |
| `method-pattern` | 方法名称，支持*模糊匹配，不指定时列出当前类下所有的方法      |

**使用示例**

```shell
[arthas@73802]$ sm -d cn.probiecoder.arthas_demo.TestController throwException
 declaring-class  cn.probiecoder.arthas_demo.TestController                                             
 method-name      throwException                                                                        
 modifier         public                                                                                
 annotation       org.springframework.web.bind.annotation.GetMapping                                    
 parameters                                                                                             
 return           void                                                                                  
 exceptions                                                                                             
 classLoaderHash  76ed5528 
 
// 列出所有方法
[arthas@73802]$ sm cn.probiecoder.arthas_demo.TestController 
cn.probiecoder.arthas_demo.TestController <init>()V
cn.probiecoder.arthas_demo.TestController throwException()V
......
```



### 14、stack

输出当前方法被调用的调用路径

**语法**

```shell
stack class-pattern [method-pattern] [condition-express] 
```



**参数说明**

| 参数名称            | 参数说明                                                     |
| ------------------- | ------------------------------------------------------------ |
| `class-pattern`     | 类路径                                                       |
| `method-pattern`    | 监控方法，可选，为空时可以跟踪类下所有方法                   |
| `condition-express` | 条件表达式，一个合法的`ognl`表达式，可以参考核心变量，`#cost`：执行时间，使用条件表达式时，监控方法需要指定 |

**使用示例**

```shell
// 按照时间过滤
stack cn.probiecoder.arthas_demo.TestController longRunning '#cost>2000'

// 匹配第一个参数的长度为3
stack cn.probiecoder.arthas_demo.TestController hello params[0].length==3
```



### 15、trace

方法内部调用路径，并输出方法路径上的每个节点上耗时，可以用于分析方法执行性能瓶颈

**语法**

```shell
trace [-n <value>] [--skipJDKMethod <value>] class-pattern method-pattern [condition-express]
```



**参数说明**

| 参数名称            | 参数说明                                                    |
| ------------------- | ----------------------------------------------------------- |
| `-n`                | 指定捕捉结果的次数，达到限制次数时直接停止监听              |
| `--skipJDKMethod`   | 是否包含 jdk 里的函数调用，默认`false`                      |
| `class-pattern`     | 类路径                                                      |
| `method-pattern`    | 方法名，支持*通配匹配                                       |
| `condition-express` | 过滤条件表达式，符合`ognl`的表达式，支持`#cost`进行时间过滤 |

**使用示例**

```shell
// 启用jdk方法调用监控，限制监听次数
[arthas@73802]$ trace --skipJDKMethod true -n 1 cn.probiecoder.arthas_demo.TestController longRunning '#cost>2000'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 35 ms, listenerId: 20
`---ts=2025-05-08 01:16:40.324;thread_name=http-nio-8080-exec-2;id=46;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@5e8507f1
    `---[2200.492615ms] cn.probiecoder.arthas_demo.TestController:longRunning()

Command execution times exceed limit: 1, so command will exit. You can set it with -n option.

```



### 16、watch

函数执行数据观测，能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看

**语法**

```shell
watch [-b] [-e] [-f] [-s] [-n <value>] class-pattern method-pattern [express] [condition-express]  
```



**参数说明**

| 参数名称               | 参数说明                                                     |
| ---------------------- | ------------------------------------------------------------ |
| `-b`                   | 在**函数调用之前**观察，由于观察事件点是在函数调用前，此时返回值或异常均不存在 |
| `-e`                   | 在**函数异常之后**观察                                       |
| `-f`                   | 在**函数结束之后**(正常返回和异常返回)观察，默认值           |
| `-s`                   | 在**函数成功返回之后**观察                                   |
| `-n`                   | 执行监控次数                                                 |
| `class-pattern`        | 要监控的类路径                                               |
| `method-pattern`       | 方法匹配模式，可以通过*进行通配匹配                          |
| `express`              | 观察表达式，默认值：`{params, target, returnObj}`，围绕`Advice`通知对象，`get`请求使用`params`观察时入参为空，需要使用`params[0]` |
| `condition-expression` | 条件表达式，合法的`ognl`表达式，根据条件过滤拦截             |

**使用示例**

```shell
// 方法调用前监听
[arthas@73802]$ watch -b cn.probiecoder.arthas_demo.TestController hello {params[0],returnObj}
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 28 ms, listenerId: 28
method=cn.probiecoder.arthas_demo.TestController.hello location=AtEnter
ts=2025-05-08 01:38:32.377; [cost=0.006553ms] result=@ArrayList[
    @String[waw],
    null,

// 限制监听次数
[arthas@73802]$ watch -b -n 1 cn.probiecoder.arthas_demo.TestController hello {params[0],returnObj}
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 38 ms, listenerId: 29
method=cn.probiecoder.arthas_demo.TestController.hello location=AtEnter
ts=2025-05-08 01:41:43.892; [cost=0.022054ms] result=@ArrayList[
    @String[waw],
    null,
]
Command execution times exceed limit: 1, so command will exit. You can set it with -n option.

```



## 三、后台任务

需要监控偶发性行为时，可以借助后台任务捕获，同时将捕获结果可以输出到文件

### 1、执行后台任务

在命令最后边添加 **`*`** 执行后台任务，命令不会阻断终端屏幕，可以继续执行其他任务

```java
trace Class method &
```



### 2、查看后台任务

```java
jobs
```

```java
[arthas@252]$ jobs
[10]*
       Running           trace org.apache.aries.spifly.Util  storeContextClassloader &
       execution count : 0
       start time      : Mon May 12 19:33:48 CST 2025
       timeout date    : Tue May 13 19:33:48 CST 2025
       session         : eb3cedf4-75a1-488c-af93-df3405ea7398 (current)
```

结果会输出任务id、任务状态，已执行捕获次数等信息



### 3、停止命令

异步执行的命令，如果希望停止，可执行`kill <job-id>



### 4、任务输出重定向

可通过`>`或者`>>`将任务输出结果输出到指定的文件中，可以和`&`一起使用，实现 arthas 命令的后台异步任务

```shell
$ trace Test t >> test.out &
```



## 四、附录

官方文档：[https://arthas.aliyun.com/doc/quick-start.html](https://arthas.aliyun.com/doc/quick-start.html)

OGNL表达式官网：[https://commons.apache.org/dormant/commons-ognl/language-guide.html](https://commons.apache.org/dormant/commons-ognl/language-guide.html)

命令参数：`[xx]`选项为可选参数	

