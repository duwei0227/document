---
layout: post
title: Grafana安装与基础配置
categories: [Java]
permalink: java/virtual_thread.html
---

虚拟线程是一种比传统 Java 线程（OS线程）更轻量的线程实现。与传统线程由操作系统管理不同，虚拟线程由 JVM 管理，多个虚拟线程可以在少量的操作系统线程（平台线程）上运行。虚拟线程的创建、切换和销毁的开销远小于传统线程，同时虚拟线程可以和普通线程一样的使用。



#### 1、虚拟线程的创建

##### 1.1 构建 Thread.Builder

```java
Thread.Builder builder = Thread.ofVirtual().name("worker-", 0);    
```

`name`有2个不同参数列表的实现

* `name(String *name*)`指定虚拟线程的名称，后期通过`builder`启动的虚拟线程名称都是相同的
* `name(String *prefix*, long *start*)`指定虚拟线程的`prefix`和开始线程的索引



##### 1.2 使用Builder启动虚拟线程

```java
Thread thread = builder.start();
```

由于虚拟线程也是通过`Thread`创建产生，当`Thread`实例创建以后，后续的使用和普通线程是相同的



##### 1.3 未捕获异常处理

```java
builder.uncaughtExceptionHandler((t, e) -> {
            System.out.println(t.getName() + "抛出了异常" + e.getMessage());
        });
```



#### 2、虚拟线程栈查看

```shell
jcmd <PID> Thread.dump_to_file -format=text <file>
jcmd <PID> Thread.dump_to_file -format=json <file>
```

输出示例

```json
{
   "tid": "20",
   "name": "vt-worker-0",
   "stack": [
      "java.base\/java.lang.VirtualThread.parkNanos(VirtualThread.java:635)",
      "java.base\/java.lang.VirtualThread.sleepNanos(VirtualThread.java:807)",
      "java.base\/java.lang.Thread.sleep(Thread.java:556)",
      "java.base\/java.util.concurrent.TimeUnit.sleep(TimeUnit.java:446)",
      "cn.probiecoder.concurrency.VirtualThread.lambda$0(VirtualThread.java:15)",
      "java.base\/java.lang.VirtualThread.run(VirtualThread.java:329)"
   ]
 }
```



**使用`jcmd <PID> Thread.print`直接输出线程栈信息时无详细的虚拟线程信息**



#### 3、线程数的设置

可以通过如下 `System.property`设置`ForkJoinPool`的参数

```java
jdk.virtualThreadScheduler.parallelism
jdk.virtualThreadScheduler.maxPoolSize
jdk.virtualThreadScheduler.minRunnable
```





#### 3、底层实现

在执行`start()`方法时会创建正式的虚拟线程，并指定虚拟线程名称，所依托的普通线程调度器`scheduler`

`ThreadBuilder.VirtualThreadBuilder`

```java
public Thread unstarted(Runnable task) {
    Objects.requireNonNull(task);
    var thread = newVirtualThread(scheduler, nextThreadName(), characteristics(), task);
    UncaughtExceptionHandler uhe = uncaughtExceptionHandler();
    if (uhe != null)
        thread.uncaughtExceptionHandler(uhe);
    return thread;
}
```



`VirtualThread`继承`sealed`类`BaseVirtualThread`，继承`Thread`

```java
VirtualThread(Executor scheduler, String name, int characteristics, Runnable task) {
    super(name, characteristics, /*bound*/ false);
    Objects.requireNonNull(task);

    // choose scheduler if not specified
    if (scheduler == null) {
        Thread parent = Thread.currentThread();
        if (parent instanceof VirtualThread vparent) {
            scheduler = vparent.scheduler;
        } else {
            scheduler = DEFAULT_SCHEDULER;
        }
    }

    this.scheduler = scheduler;
    this.cont = new VThreadContinuation(this, task);
    this.runContinuation = this::runContinuation;
}
```



默认的调度器为`ForkJoinPool`

```java
private static final ForkJoinPool DEFAULT_SCHEDULER = createDefaultScheduler();

private static ForkJoinPool createDefaultScheduler() {
    ForkJoinWorkerThreadFactory factory = pool -> {
        PrivilegedAction<ForkJoinWorkerThread> pa = () -> new CarrierThread(pool);
        return AccessController.doPrivileged(pa);
    };
    PrivilegedAction<ForkJoinPool> pa = () -> {
        int parallelism, maxPoolSize, minRunnable;
        String parallelismValue = System.getProperty("jdk.virtualThreadScheduler.parallelism");
        String maxPoolSizeValue = System.getProperty("jdk.virtualThreadScheduler.maxPoolSize");
        String minRunnableValue = System.getProperty("jdk.virtualThreadScheduler.minRunnable");
        if (parallelismValue != null) {
            parallelism = Integer.parseInt(parallelismValue);
        } else {
            parallelism = Runtime.getRuntime().availableProcessors();
        }
        if (maxPoolSizeValue != null) {
            maxPoolSize = Integer.parseInt(maxPoolSizeValue);
            parallelism = Integer.min(parallelism, maxPoolSize);
        } else {
            maxPoolSize = Integer.max(parallelism, 256);
        }
        if (minRunnableValue != null) {
            minRunnable = Integer.parseInt(minRunnableValue);
        } else {
            minRunnable = Integer.max(parallelism / 2, 1);
        }
        Thread.UncaughtExceptionHandler handler = (t, e) -> { };
        boolean asyncMode = true; // FIFO
        return new ForkJoinPool(parallelism, factory, handler, asyncMode,
                     0, maxPoolSize, minRunnable, pool -> true, 30, SECONDS);
    };
    return AccessController.doPrivileged(pa);
}
```



将虚拟线程绑定到`PlatformThread`

```java
@ChangesCurrentThread
@ReservedStackAccess
private void mount() {
    // notify JVMTI before mount
    notifyJvmtiMount(/*hide*/true);

    // sets the carrier thread
    Thread carrier = Thread.currentCarrierThread();
    setCarrierThread(carrier);

    // sync up carrier thread interrupt status if needed
    if (interrupted) {
        carrier.setInterrupt();
    } else if (carrier.isInterrupted()) {
        synchronized (interruptLock) {
            // need to recheck interrupt status
            if (!interrupted) {
                carrier.clearInterrupt();
            }
        }
    }

    // set Thread.currentThread() to return this virtual thread
    carrier.setCurrentThread(this);
}
```

