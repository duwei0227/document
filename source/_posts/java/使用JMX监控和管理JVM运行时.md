---
layout: post
title: 使用JMX监控和管理JVM运行时
categories: [Java]
permalink: java/jmx.html
---



### 一、BufferBoolMXBean

**JVM 的 BufferPool 是一种用于管理直接缓冲区 (Direct Buffer) 的资源池**，它是由 Java 平台引入来优化直接内存分配的机制，直接缓冲区通常用于 I/O 操作（如文件或网络通信）以提高性能，**内存区域不统计在`JVM`堆内存中**。



**主要方法：**

| 名称                     | 方法               | 描述                                             |
| ------------------------ | ------------------ | ------------------------------------------------ |
| 名称 (Name)              | `getName`          | BufferPool 的名称，例如 `"direct"` 和 `"mapped"` |
| 缓冲区数量 (Count)       | `getCount`         | 当前池中活动缓冲区的数量                         |
| 内存使用量 (Memory Used) | `getMemoryUsed`    | 池中分配的直接内存的总量（以字节为单位）         |
| 总容量 (Total Capacity)  | `getTotalCapacity` | 池中所有缓冲区的总容量（以字节为单位）           |



`Memory Used`和`Total Capacity`的区别：

`Memory Used`:

* 表示当前 **已经分配** 的内存总量
* 包括所有已创建缓冲区的内存大小，即正在被使用的直接内存

`Total Capacity`:

* 表示所有缓冲区的 **总容量**
* 包括分配的缓冲区的逻辑容量，而不仅是实际使用的内存



| 属性               | 含义                     | 包含内容               | 变化情况                     |
| ------------------ | ------------------------ | ---------------------- | ---------------------------- |
| **Memory Used**    | 当前已分配的直接内存总量 | 实际分配的内存大小     | 缓冲区分配增加，释放减少     |
| **Total Capacity** | 所有缓冲区的逻辑容量总和 | 缓冲区声明的容量总大小 | 缓冲区分配时增加，释放时减少 |



**常见的`BufferPool`：**

| 名称     | 描述               | 分配方式                                                     |
| -------- | ------------------ | ------------------------------------------------------------ |
| `direct` | 直接内存缓冲区     | `ByteBuffer.allocateDirect()`                                |
| `mapped` | 映射到文件的缓冲区 | `FileChannel.map()` `MappedByteBuffer buffer = channel.map(FileChannel.MapMode.*READ_WRITE*, 0, 1024)` |



**直接内存上限管理：**

直接内存的上限由 JVM 参数 `-XX:MaxDirectMemorySize` 决定，如果没有显式设置，默认值与堆大小 (`-Xmx`) 相同；

如果在分配直接内存时，内存空间不足会抛出 `java.lang.OutOfMemoryError: Direct buffer memory`异常



**监控**

* 使用图形工具，例如 `VisualVM`(需要安装`VisualVM-BufferMonitor`插件)
* 使用`jcmd`监控`Native Memory`的使用
  * 启用直接内存跟踪 `-XX:NativeMemoryTracking=options`,`options`支持`summary`：提供总的内存使用统计，开销较低，`detail`：提供详细的内存分配信息，开销较高；一般启用`summary`就可以
  * 使用`jcmd pid VM.native_memory [summary|detail]`查看内存分布情况，直接内存一般在`Direct`或`Internal(jdk1.8)`模块下，可以使用`jcmd pid VM.native_memory baseline`创建数据基线，然后通过`jcmd pid VM.native_memory [summary|detail].diff`和基线比较，确定内存对比基线的升降



**问题定位**

* 发生`OutOfMemoryError: Direct buffer memory`：分析异常堆栈信息，找到导致`OOM`的位置
* 直接内存占比较高：

参考文档：

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/BufferPoolMXBean.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/BufferPoolMXBean.html)





### 二、ClassLoadingMXBean

提供有关类加载器行为的监控和管理功能



 **主要方法**

| 方法                         | 描述                                                     |
| ---------------------------- | -------------------------------------------------------- |
| `getLoadedClassCount()`      | 返回当前加载到 JVM 中的类的数量。                        |
| `getTotalLoadedClassCount()` | 返回 JVM 自启动以来加载的类的总数（包括卸载的类）。      |
| `getUnloadedClassCount()`    | 返回 JVM 自启动以来卸载的类的总数。                      |
| `isVerbose()`                | 检查是否启用了类加载详细信息。                           |
| `setVerbose(boolean value)`  | 启用或禁用类加载详细信息（仅限调试用途，可能影响性能）。 |



启用`verbose`详细输出时，会在日志中输出如下内容：

`[Loaded 类全路径 from file:jar包路径]`

```java
[Loaded io.lettuce.core.event.metrics.CommandLatencyEvent from file:/E:/pub/repos/io/lettuce/lettuce-core/5.0.5.RELEASE/lettuce-core-5.0.5.RELEASE.jar]

```



**使用示例**

```java
// ClassLoadingMXBean mxBean = ManagementFactory.getClassLoadingMXBean();
ClassLoadingMXBean mxBean = ManagementFactory.getPlatformMXBean(ClassLoadingMXBean.class);

sb.append("TotalLoadedClassCount: ").append(mxBean.getTotalLoadedClassCount()).append("\t")
        .append("LoadedClassCount: ").append(mxBean.getLoadedClassCount()).append("\t")
        .append("UnloadedClassCount: ").append(mxBean.getUnloadedClassCount()).append("\t")
        .append("isVerbose: ").append(mxBean.isVerbose()).append("\n");

if (!mxBean.isVerbose()) {
    mxBean.setVerbose(true);
}
```



### 三、CompilationMXBean

用于监控和管理 JVM 编译器的运行时行为，它提供有关即时编译（JIT，Just-In-Time）编译器的性能统计和配置信息



**主要方法**

| 方法                                     | 描述                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `getName()`                              | 获取 JIT 编译器的名称。                                      |
| `isCompilationTimeMonitoringSupported()` | 检查是否支持编译时间监控（某些 JVM 实现可能不支持）。        |
| `getTotalCompilationTime()`              | 返回自 JVM 启动以来 JIT 编译花费的总时间（以毫秒为单位）。如果使用多线程编译，该值为每个线程编译时间总和 |



**示例**

```java
CompilationMXBean compilationMXBean = ManagementFactory.getCompilationMXBean();
sb.append("isCompilationTimeMonitoringSupported: ").append(compilationMXBean.isCompilationTimeMonitoringSupported()).append("\t")
        .append("getName: ").append(compilationMXBean.getName()).append("\t")
        .append("getTotalCompilationTime: ").append(compilationMXBean.getTotalCompilationTime()).append("\n");
```





**配置参数**

| 参数                                 | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `-XX:+TieredCompilation`             | 启用分层编译（默认开启）。                                   |
| `-XX:CompileThreshold=<invocations>` | 方法从解释执行到 JIT 编译所需的调用次数（默认约为 10,000 次）。 |
|                                      |                                                              |

 **与编译器日志相关的参数**

| 参数                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `-XX:+PrintGC`        | 打印 GC 信息（结合编译时间监控分析编译与 GC 的关系）。       |
| `-XX:+LogCompilation` | 将 JIT 编译日志输出到文件，生成可视化日志（配合工具 `JITWatch` 使用）。 |
| `-XX:+PrintCodeCache` | 打印 JIT 编译器的代码缓存使用信息。                          |
| `-XX:+CITime`         | 输出编译器的时间统计信息（包括每个编译线程的耗时）。         |

------

 **与代码缓存（Code Cache）相关的参数**

| 参数                                | 描述                                   |
| ----------------------------------- | -------------------------------------- |
| `-XX:InitialCodeCacheSize=<size>`   | 设置代码缓存的初始大小（默认 32MB）。  |
| `-XX:ReservedCodeCacheSize=<size>`  | 设置代码缓存的最大大小（默认 240MB）。 |
| `-XX:+UseCodeCacheFlushing`         | 启用代码缓存溢出时的缓存清理机制。     |
| `-XX:CodeCacheExpansionSize=<size>` | 设置代码缓存增长的块大小。             |





**内存使用监控**

使用 `jcmd pid VM.native_memory summary`命令查看`native_memory`内存使用情况

* `Compiler`:JIT编译器使用到的内存
* `Code`:JIT编译器编译缓存代码占用空间

```java

Native Memory Tracking:

Total: reserved=2377592KB, committed=1131948KB

-           Code (reserved=258133KB, committed=46169KB)
                (malloc=8533KB #17365) 
                (mmap: reserved=249600KB, committed=37636KB)

-           Compiler (reserved=365KB, committed=365KB)
                (malloc=234KB #638)
                (arena=131KB #7)
```



### 四、GarbageCollectorMXBean

用于监控和管理 Java 虚拟机中垃圾收集器的行为和性能



**主要方法**

| 方法                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `getName()`            | 返回垃圾收集器的名称                                         |
| `getCollectionCount()` | 返回垃圾收集器运行的总次数。如果无法获取信息，返回 `-1`      |
| `getCollectionTime()`  | 返回垃圾收集器运行的总耗时（以毫秒为单位）。如果无法获取信息，返回 `-1` |
| `getMemoryPoolNames()` | 返回与垃圾收集器关联的内存池名称                             |



**使用示例**

```java
List<GarbageCollectorMXBean> mxBeans = ManagementFactory.getGarbageCollectorMXBeans();
mxBeans.forEach(mxBean -> {
    sb.append("Name: ").append(mxBean.getName()).append("\t")
            .append("CollectionCount: ").append(mxBean.getCollectionCount()).append("\t")
            .append("CollectionTime: ").append(mxBean.getCollectionTime()).append("\t")
            .append("MemoryPoolNames: ").append(Arrays.toString(mxBean.getMemoryPoolNames())).append("\n");
});
```



### 五、 MemoryManagerMXBean

用于管理和监控与 JVM 内存管理相关的组件。它是内存管理的高层接口，可以用于查看 JVM 中的内存管理器及其关联的内存池。



**主要方法**

| 方法                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `getName()`            | 返回内存管理器的名称                                         |
| `getMemoryPoolNames()` | 返回与该内存管理器关联的内存池名称                           |
| `isValid()`            | 返回内存管理器是否仍然有效 如果内存管理器已被 JVM 停用或不再使用，则返回 `false`，否则返回 `true` |



**使用示例**

```java
List<MemoryManagerMXBean> mxBeans = ManagementFactory.getMemoryManagerMXBeans();
mxBeans.forEach(mxBean -> {
    sb.append("Name: ").append(mxBean.getName()).append("\t")
            .append("isValid: ").append(mxBean.isValid()).append("\t")
            .append("MemoryPoolNames: ").append(Arrays.toString(mxBean.getMemoryPoolNames())).append("\n");
});
```





### 六、MemoryPoolMXBean

用于监控 JVM 中的内存池。内存池是 JVM 内存的逻辑划分，例如 Eden、Survivor、Old Gen、Metaspace 等。

通过 `MemoryPoolMXBean`，可以获取内存池的使用情况、配置参数以及与垃圾收集器和内存管理器的关联信息



**主要方法**

| 方法                                    | 描述                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| `getName()`                             | 返回内存池名称                                               |
| `getType()`                             | 返回内存池类型 `heap` `nonheap`                              |
| `getMemoryManageNames()`                | 返回管理此内存池的内存管理器的名称                           |
| `isValid()`                             | 测试此内存池在 Java 虚拟机中是否有效                         |
| `getUsage()`                            | 返回此内存池的内存使用量估计值。`null` 如果此内存池无效（即不再存在），则此方法返回 |
| `isUsageThresholdSupported()`           | 测试此内存池是否支持使用阈值，**需要先判断是否支持在获取相关`threshold`指标** |
| `getUsageThreshold()`                   | 返回此内存池的使用阈值（以字节为单位）                       |
| `getUsageThresholdCount()`              | 返回内存池使用量超过阈值的次数                               |
| `setUsageThreshold()`                   | 设置使用率阈值（长阈值）                                     |
| `isUsageThresholdExceeded()`            | 测试此内存池的内存使用量是否达到或超过其使用量阈值           |
| `getCollectionUsage()`                  | 获取垃圾收集后内存池的使用信息，返回 `MemoryUsage` 对象      |
| `isCollectionUsageThresholdSupported()` | 判断当前内存池是否支持设置 `CollectionUsageThreshold`，需要先判断是否支持在获取相关`threshold`指标 |
| `getCollectionUsageThreshold()`         | 返回当前设置的垃圾收集后使用阈值                             |
| `getCollectionUsageThresholdCount()`    | 返回垃圾收集后内存池使用量超过阈值的次数                     |
| `isCollectionUsageThresholdExceeded()`  | 检查垃圾收集后内存池的使用量是否达到或超过设置的 `CollectionUsageThreshold`（垃圾收集后使用阈值） |
| `setCollectionUsageThreshold()`         | 设置垃圾收集后内存池的使用阈值，单位为字节                   |
| `getPeakUsage()`                        | 返回自 Java 虚拟机启动或峰值重置以来此内存池的峰值内存使用量。`null` 如果此内存池无效（即不再存在），则此方法返回。 |
| `resetPeakUsage()`                      | 将此内存池的峰值内存使用量统计信息重置为当前内存使用量       |



可以考虑设置合理的内存`Threshold`，及时监控内存各个区使用情况



**使用示例**

```java
List<MemoryPoolMXBean> mxBeans = ManagementFactory.getMemoryPoolMXBeans();
mxBeans.forEach(mxBean -> {
    sb.append("Name: ").append(mxBean.getName()).append("\t")
            .append("Type: ").append(mxBean.getType().toString()).append("\t")
            .append("MemoryManageNames: ").append(Arrays.toString(mxBean.getMemoryManagerNames())).append("\t")
            .append("isValid: ").append(mxBean.isValid()).append("\n")
            .append("Usage: ").append(mxBean.getUsage()).append("\t");
    if (mxBean.isUsageThresholdSupported()) {
        sb.append("isUsageThresholdSupported: ").append(mxBean.isUsageThresholdSupported()).append("\t")
                .append("UsageThreshold: ").append(mxBean.getUsageThreshold()).append("\t")
                .append("UsageThresholdCount: ").append(mxBean.getUsageThresholdCount()).append("\t")
                .append("isUsageThresholdExceeded: ").append(mxBean.isUsageThresholdExceeded()).append("\n");
    }

    sb.append("CollectionUsage: ").append(mxBean.getCollectionUsage()).append("\t");
    if (mxBean.isCollectionUsageThresholdSupported()) {
        sb.append("isCollectionUsageThresholdSupported: ").append(mxBean.isCollectionUsageThresholdSupported()).append("\t")
                .append("CollectionUsageThreshold: ").append(mxBean.getCollectionUsageThreshold()).append("\t")
                .append("CollectionUsageThresholdCount: ").append(mxBean.getCollectionUsageThresholdCount()).append("\t")
                .append("isCollectionUsageThresholdExceeded: ").append(mxBean.isCollectionUsageThresholdExceeded()).append("\n");
    }

    sb.append("PeakUsage: ").append(mxBean.getPeakUsage()).append("\n");

});
```



**文档**

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/MemoryPoolMXBean.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/MemoryPoolMXBean.html)





### 七、MemoryMXBean

获取堆内存和非堆内存的使用信息，并与垃圾回收交互



**主要方法**

| 方法                      | 描述                 |
| ------------------------- | -------------------- |
| `gc()`                    | 运行垃圾回收         |
| `getHeapMemoryUsage()`    | 获取堆内存使用数据   |
| `getNonHeapMemoryUsage()` | 获取非堆内存使用数据 |
| `isVerbose()`             | 当前是否启用详细输出 |
| `setVerbose()`            | 启用或禁用详细输出   |



非堆内存：

* 所有线程共享区域



**使用示例**

```Java
MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
sb.append("HeapMemoryUsage: ").append(memoryMXBean.getHeapMemoryUsage()).append("\n")
        .append("NonHeapMemoryUsage: ").append(memoryMXBean.getNonHeapMemoryUsage()).append("\n")
        .append("isVerbose: ").append(memoryMXBean.isVerbose()).append("\n");
```



**文档**

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/MemoryMXBean.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/MemoryMXBean.html)



**内存MXBean对比**

| 功能/接口            | MemoryMXBean                        | MemoryPoolMXBean                   | MemoryManagerMXBean          |
| -------------------- | ----------------------------------- | ---------------------------------- | ---------------------------- |
| **作用范围**         | 整个 JVM 的堆内存和非堆内存         | 单个内存池（如 Eden、Old Gen 等）  | 内存管理器（如垃圾收集器）   |
| **核心功能**         | 总览 JVM 内存使用情况；垃圾回收控制 | 监控特定内存池的使用情况和阈值     | 管理内存池的内存管理器信息   |
| **是否支持阈值**     | 否                                  | 是                                 | 否                           |
| **与垃圾回收的关系** | 显式触发垃圾回收                    | 监控垃圾回收后内存池的使用情况     | 管理负责垃圾回收的内存池     |
| **使用场景**         | 监控 JVM 整体内存，调试和性能优化   | 监控内存池使用量，分析内存分配问题 | 分析垃圾收集器与内存池的关系 |





### 八、ThreadMXBean

用于获取关于线程的信息、线程状态、线程性能和线程死锁的诊断



**主要方法**

| 方法                                     | 描述                                              |
| ---------------------------------------- | ------------------------------------------------- |
| `getThreadCount()`                       | 获取当前活动线程总数，包含非`daemon`线程          |
| `getPeakThreadCount()`                   | 获取从`JVM`启动到当前峰值线程总数                 |
| `getDaemonThreadCount()`                 | 获取当前活动的`daemon`线程                        |
| `findDeadlockedThreads()`                | 死锁检测                                          |
| `getAllThreadIds()`                      | 获取所有活动线程id                                |
| `getThreadInfo(long id, int maxDepth)`   | 根据线程id获取线程信息,`maxDepth`指定堆栈跟踪深度 |
| `getThreadInfo(long[] id, int maxDepth)` | 根据线程id获取线程信息,`maxDepth`指定堆栈跟踪深度 |



**ThreadInfo主要方法**

| 方法               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `getThreadName()`  | 线程名称                                                     |
| `getThreadId()`    | 线程ID                                                       |
| `getThreadState()` | 线程状态，`NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATE` |
| `getBlockCount()`  | 线程因进入或重新进入监视器而受阻的总次数                     |
| `getBlockTime()`   | 线程因进入或重新进入监视器而受阻的总次数累计耗用时间（毫秒为单位） |
| `getWaitedCount()` | 线程等待通知的总次数                                         |
| `getWaitedTime()`  | 线程等待通知的累计耗用时间（毫秒为单位）                     |
| `isDaemon()`       | 线程是否为`daemon`线程                                       |
| `getPriority()`    | 获取线程优先级                                               |
| `getLockName()`    |                                                              |
| `getStackTrace()`  | 获取堆栈信息，在获取`ThreadInfo`时，需要指定`maxDepth`       |



**使用示例**

```java
ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
sb.append("ThreadCount: ").append(threadMXBean.getThreadCount()).append("\n");
sb.append("PeakThreadCount: ").append(threadMXBean.getPeakThreadCount()).append("\n");
sb.append("DaemonThreadCount: ").append(threadMXBean.getDaemonThreadCount()).append("\n");
sb.append("DeadlockedThreads: ").append(Arrays.toString(threadMXBean.findDeadlockedThreads())).append("\n");

long[] threadIds = threadMXBean.getAllThreadIds();
ThreadInfo[] threadInfos = threadMXBean.getThreadInfo(threadIds, 10);  // 不指定 maxDepth 时，无堆栈信息
Arrays.stream(threadInfos)
        .filter(threadInfo -> threadInfo.getThreadName().equals("Thread-117")).findFirst()
        .ifPresent(threadInfo -> {
            sb.append("Name: ").append(threadInfo.getThreadName()).append("\n");
            sb.append("Id: ").append(threadInfo.getThreadId()).append("\n");
            sb.append("LockName: ").append(threadInfo.getLockName()).append("\n");
            sb.append("State: ").append(threadInfo.getThreadState()).append("\n");
            sb.append("Block: ").append(threadInfo.getBlockedCount()).append("\n");
            sb.append("Waited: ").append(threadInfo.getWaitedCount()).append("\n");
            sb.append("StackTrace: ").append("\n");
            StackTraceElement[] stackTraceElements = threadInfo.getStackTrace();
            System.out.println(stackTraceElements.length);
            for (StackTraceElement stackTraceElement : stackTraceElements) {
                sb.append(stackTraceElement.toString()).append("\n");
            }
        });
```





**文档**

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/ThreadMXBean.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/ThreadMXBean.html)	

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/ThreadInfo.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/ThreadInfo.html)

[https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.State.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.State.html)





### 九、OperatingSystemMXBean

用于从操作系统获取相关的系统级指标



**主要方法**

| 方法                       | 描述                                                   |
| -------------------------- | ------------------------------------------------------ |
| `getName()`                | 获取系统名称                                           |
| `getVersion()`             | 获取系统版本                                           |
| `getArch()`                | 获取系统架构                                           |
| `getAvailableProcessors()` | 获取虚拟机可用的处理器数量                             |
| `getSystemLoadAverage()`   | 返回最后一分钟的系统负载平均值，如果不允许获取返回负数 |



**com.sun.management.OperatingSystemMXBean主要方法**

| 方法                              | 描述               |
| --------------------------------- | ------------------ |
| `getTotalPhysicalMemorySize()`    | 系统总物理内存     |
| `getFreeePhysicalMemorySize()`    | 系统空闲物理内存   |
| `getTotalSwapSpaceSize()`         | 总交换空间         |
| `getFreeSwapSpaceSize()`          | 空闲交换空间       |
| `getSystemCpuLoad()`              | 系统CPU使用率      |
| `getProcessCpuLoad()`             | JVM进程CPU使用率   |
| `getProcessCpuTime()`             | JVM进程CPU处理时间 |
| `getCommittedVirtualMemorySize()` | 已提交的虚拟内存   |

`com.sun.management.OperatingSystemMXBean`继承自`java.lang.management.OperatingSystemMXBean`，通过`ManagementFactory.getOperatingSystemMXBean()`获取以后需要进行类型转换；`com.sun.management.OperatingSystemMXBean` 是 Sun/Oracle JVM 的实现，其他 JVM（如 OpenJ9）可能不支持



**使用示例**

```java
OperatingSystemMXBean os = ManagementFactory.getOperatingSystemMXBean();
sb.append("Name: ").append(os.getName()).append("\n");
sb.append("Version: ").append(os.getVersion()).append("\n");
sb.append("Arch: ").append(os.getArch()).append("\n");
sb.append("Available Processors: ").append(os.getAvailableProcessors()).append("\n");
sb.append("System LoadAverage: ").append(os.getSystemLoadAverage()).append("\n");

com.sun.management.OperatingSystemMXBean sunOs = (com.sun.management.OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();
sb.append("Total PhysicalMemorySize: ").append(sunOs.getTotalPhysicalMemorySize()).append("\n");
sb.append("Free PhysicalMemorySize: ").append(sunOs.getFreePhysicalMemorySize()).append("\n");
sb.append("Total Swap Space: ").append(sunOs.getTotalSwapSpaceSize()).append("\n");
sb.append("Free Swap Space: ").append(sunOs.getFreeSwapSpaceSize()).append("\n");
sb.append("System Cpu Load: ").append(sunOs.getSystemCpuLoad()).append("\n");
sb.append("Process Cpu Load: ").append(sunOs.getProcessCpuLoad()).append("\n");
sb.append("Process Cpu Time: ").append(sunOs.getProcessCpuTime()).append("\n");
sb.append("Committed Virtual Memory Size: ").append(sunOs.getCommittedVirtualMemorySize()).append("\n");
```



**文档**

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/OperatingSystemMXBean.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/OperatingSystemMXBean.html)



### 十、RuntimeMXBean

取与 Java 虚拟机（JVM）运行时相关的信息和属性



**主要方法**


| **方法名称**                  | **描述**                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| `getName()`                  | 返回 JVM 名称，格式为 `<pid>@<hostname>`，表示 JVM 的进程 ID 和主机名。 |
| `getStartTime()`             | 返回 JVM 的启动时间，表示从纪元开始的毫秒数（Unix 时间戳）。 |
| `getUptime()`                | 返回 JVM 自启动以来的运行时间（以毫秒为单位）。              |
| `getVmName()`                | 具体的JVM名称，例如 `Java HotSpot(TM) 64-Bit Server VM`。    |
| `getVmVendor()`              | JVM虚拟机供应商名称，例如 `Oracle Corporation`。             |
| `getVmVersion()`             | JVM虚拟机版本号，例如 `25.201-b09`。                         |
| `getSpecName()`              | Java虚拟机规范名称，例如 `Java Virtual Machine Specification`。 |
| `getSpecVendor()`            | Java虚拟机规范的供应商名称，例如 `Oracle Corporation`。      |
| `getSpecVersion()`           | Java虚拟机规范版本号，例如 `1.8` 或 `11`。                   |
| `getInputArguments()`        | 传递给 Java 虚拟机的输入参数，其中不包括主方法的参数。例如 `-Xms512m` 或 `-Xmx1024m`。 |
| `getSystemProperties()`      | 返回 JVM 的系统属性键值对，例如操作系统信息、Java 版本等。   |
| `getBootClassPath()`         | 返回引导类加载器的类路径（仅在特定 JVM 上支持）。            |
| `isBootClassPathSupported()` | 检查当前 JVM 是否支持 `getBootClassPath()` 方法。            |
| `getClassPath()`             | 返回类路径字符串，表示 `-classpath` 或 `-cp` 参数指定的值。  |
| `getLibraryPath()`           | 返回动态库加载路径字符串，表示 `-Djava.library.path` 参数的值。 |
| `getManagementSpecVersion()` | 返回 JMX 规范版本，例如 `1.2`。                              |
| `getPid()`                   | 返回当前 JVM 的进程 ID（PID）。 (Java 9+)                    |



**示例**

```java
RuntimeMXBean mxBean = ManagementFactory.getRuntimeMXBean();
sb.append("Name: ").append(mxBean.getName()).append("\n");
sb.append("StartTime: ").append(mxBean.getStartTime()).append("\n");
sb.append("Uptime: ").append(mxBean.getUptime()).append("\n");
sb.append("VmName: ").append(mxBean.getVmName()).append("\n");
sb.append("VmVendor: ").append(mxBean.getVmVendor()).append("\n");
sb.append("VmVersion: ").append(mxBean.getVmVersion()).append("\n");
sb.append("IsBootClassPathSupported: ").append(mxBean.isBootClassPathSupported()).append("\n");
sb.append("BootClassPath: ").append(mxBean.getBootClassPath()).append("\n");
sb.append("LibraryPath: ").append(mxBean.getLibraryPath()).append("\n");
sb.append("ClassPath: ").append(mxBean.getClassPath()).append("\n");
sb.append("SpecName: ").append(mxBean.getSpecName()).append("\n");
sb.append("SpecVendor: ").append(mxBean.getSpecVendor()).append("\n");
sb.append("SpecVersion: ").append(mxBean.getSpecVersion()).append("\n");
sb.append("InputArguments: ").append(mxBean.getInputArguments()).append("\n");
sb.append("ManagementsSpecVersion: ").append(mxBean.getManagementSpecVersion()).append("\n");
sb.append("SystemProperties: ").append(mxBean.getSystemProperties()).append("\n");
```



**文档**

[https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/RuntimeMXBean.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/RuntimeMXBean.html)



