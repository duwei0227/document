---
title: 用 Arthas profiler 定位 Java Web 请求热点并读懂火焰图
published: true
layout: post
date: 2026-11-25
permalink: /arthas/profiler.html
categories:
  - Arthas
tags:
  - profiler
  - Java
---

## 一、什么时候考虑用 profiler

Java 应用出现 CPU 持续升高、GC 变频繁，或者请求变慢，已经从监控确认了现象，但还不知道问题集中在哪条调用链，这时就可以考虑用 Arthas 的 `profiler`。它在问题发生时采集一段调用栈，帮你找到该先看哪段代码。

| 看到的现象 | 先采什么 |
| --- | --- |
| 请求一多，CPU 就持续升高 | `cpu` · 哪条调用链最耗 CPU？ |
| GC 变频繁，想找对象从哪里创建 | `alloc` · 哪个请求路径分配量最大？ |
| 并发上来后，吞吐量上不去 | `lock` · 请求在哪个加锁点排队？ |
| 接口慢，CPU 却不高 | `wall` · 请求线程把经过时间花在哪段等待上？ |

偶发的问题，等复现时再采。如果只在高并发时出现，就让采样窗口覆盖并发高的那段时间。

## 二、profiler 语法

连上 Arthas 就可以敲 `profiler`。命令名后面跟一个动作，比如 `start` 或 `stop`，动作需要参数时再往后写。平时采一张图，先 `start`，中途用 `status` 看一眼，最后 `stop`。下面把用到的动作拆开看。

### 2.1 actions

列出当前版本支持的动作。例如，想确认当前版本有没有 `dump`，可以先跑这一条。

```bash
profiler actions
```

### 2.2 list

列出当前环境报告的采样事件。比如换到另一台机器，想确认列表里有没有 `wall`，可以用这条命令查一下。

```bash
profiler list
```

### 2.3 version

查看采样引擎 async-profiler 的版本。单独执行 `version` 查看 Arthas 自身的版本。

```bash
profiler version
```

### 2.4 start

开启一轮新采样，清空上轮累计数据。指定 `--duration` 后，到时自动停止并导出。

```bash
profiler start [--event <事件>] [--interval <间隔>] [--duration <秒数>] [--file <路径>] [--format <格式>] [--include <模式>]
```

| 选项 [短选项/长选项] | 用法与默认值 |
| --- | --- |
| `-e/--event` `<事件>` | 支持 `cpu`、`alloc`、`lock`、`wall` 等事件，用 `profiler list` 可以获取当前环境下完整的可用事件列表。默认 **`cpu`**。 |
| `-i/--interval` `<间隔>` | CPU 采样间隔，单位为纳秒。`20000000` 为 20 ms。 默认 **CPU 采样为 `10,000,000 ns`（10 ms）**。 |
| `-d/--duration` `<秒数>` | 采样时长，单位为秒。到时自动停止。 默认 **不按 `--duration` 自动停止**。 |
| `-o/--format` `<格式>` | 和 `--duration` 一起用时，指定到时自动导出的格式。未指定时可从文件后缀推断，无法推断时默认 **`html`**。可用格式见 2.8 节。 |
| `-f/--file` `<路径>` | 指定目标 JVM 可写的导出路径。 默认 **`arthas-output/` 下按时间命名的文件**（导出时）。 |
| `-I/--include` `<模式>` | 导出时保留含匹配帧的调用栈，例如 `'cn/probiecoder/*'`。与 `--duration` 一起用，放在命令末尾。默认 **不过滤**。 |

**示例：**

```bash
# 开始 CPU 采样，之后手动用 stop 结束
profiler start --event cpu

# 先结束上一轮，再按 20 ms 间隔采样 8 秒，自动导出火焰图
profiler start --event cpu --interval 20000000 --duration 8 --format html --file /tmp/cpu-8s.html

# 再单独采样 4 秒，导出时只保留经过业务包的完整调用栈
profiler start --event cpu --duration 4 --file /tmp/cpu-business.html --include 'cn/probiecoder/*'
```

### 2.5 status

查看采样是否正在运行，以及已经跑了多久。比如执行 `dump` 后，可以用它确认采样还在继续。

```bash
profiler status
```

输出：

```text
Profiling is running for 5 seconds
```

### 2.6 getSamples

查看当前累计的样本数。比如刚开始采一种事件，可以等几秒再查，看看有没有采到数据。

```bash
profiler getSamples
```


### 2.7 dump

导出已经采到的结果，采样继续进行。比如请求还在跑，想先看看图，可以用 `dump` 导出一份快照。过几秒再导出一次，第二张图还会包含第一张图里的样本。

```bash
profiler dump [--file <路径>] [--format <格式>]
```

| 选项 [短选项/长选项] | 用法与默认值 |
| --- | --- |
| `-f/--file` `<路径>` | 指定快照文件。目标 JVM 要能写入该路径。 默认 **`arthas-output/` 下按时间命名的文件**（导出时）。 |
| `-o/--format` `<格式>` | 指定快照格式，见 2.8 节「输出格式怎么选」。默认 **`html`**（未从 `--file` 后缀推断出格式时）。 |

**示例：**

```bash
profiler dump --file /tmp/cpu-snapshot.html
profiler status
```

输出：

```text
OK
Profiling is running for 5 seconds
```

### 2.8 stop

结束当前采样并导出本轮结果。比如采完 CPU 想换成 `alloc`，可以先用 `stop` 保存 CPU 图，再启动分配采样。

```bash
profiler stop [--file <路径>] [--format <格式>] [--include <模式>]
```

| 选项 [短选项/长选项] | 用法与默认值 |
| --- | --- |
| `-f/--file` `<路径>` | 指定最终文件。目标 JVM 要能写入该路径。 默认 **`arthas-output/` 下按时间命名的文件**（导出时）。 |
| `-o/--format` `<格式>` | 指定导出格式，见「输出格式怎么选」。默认 **`html`**（未从 `--file` 后缀推断出格式时）。 |
| `-I/--include` `<模式>` | 停止采样时按帧名过滤导出的调用栈，放在命令末尾。默认 **不过滤**。 |

**示例：**

```bash
profiler stop --format html --file /tmp/cpu.html
```

手动开始的那一轮，可以等到 `stop` 时再决定过滤什么。下面只保留经过示例业务包的调用栈。

**示例：**

```bash
profiler stop --format html --file /tmp/cpu-business.html --include 'cn/probiecoder/*'
```

#### 输出格式怎么选

手动采样时，在 `stop` 上选导出格式。用 `dump` 导出中途快照时也能指定格式，定时采样则在 `start --duration` 上提前指定。`jfr` 要从采样开始就配置。

| 格式 | 用途 |
| --- | --- |
| `html` / `flamegraph` | 生成可交互的 HTML 火焰图。 |
| `tree` | 按调用树查看结果。 |
| `collapsed` | 导出折叠后的文本调用栈。 |
| `flat` / `flat=N` | 按方法聚合，`N` 指定输出条数。本机分别导出了 `flat` 和 `flat=5`。 |
| `traces` / `traces=N` | 列出采样调用栈，`N` 指定输出条数。本机分别导出了 `traces` 和 `traces=5`。 |
| `md` / `md=N` | 输出 Markdown 热点报告，热点数量默认 **10**。本机分别导出了 `md` 和 `md=5`。 |
| `jfr` | 记录为 JFR，在 `start` 时指定，例如 `profiler start --format jfr --file /tmp/cpu.jfr`。 |

### 2.9 resume

在上一轮结果的基础上继续采样。比如刚用 `stop` 出了一张图，发现窗口太短，可以用 `resume` 再采几秒。恢复前后各查一次 `getSamples`，就能看到样本数有没有继续增加。

```bash
profiler resume [--event <事件>]
```

| 选项 [短选项/长选项] | 用法与默认值 |
| --- | --- |
| `-e/--event` `<事件>` | 填写上一轮使用的事件。用 `profiler list` 可以获取当前环境下完整的可用事件列表。默认 **`cpu`**。 |

**示例：**

```bash
profiler getSamples
profiler resume --event cpu
# 等几秒再查
profiler getSamples
profiler stop --file /tmp/cpu-resume.html
```

输出：

```text
4299
Profiling started
6187
OK
```

恢复采样后，样本数从 `4299` 增到 `6187`，最后导出累计火焰图。

## 三、怎么看火焰图

火焰图把一个采样窗口内的调用栈叠起来。打开 `profiler stop` 或 `profiler dump` 生成的 HTML，可以搜索类名，也可以点击某个方块，把它作为当前视图的起点，放大上方的调用分支。鼠标悬停在方块上，可以看到方法名、样本数和占全图的比例。

例如在 `alloc` 图里，点击 `OrderController.export`，就能展开看这条调用链上的分配样本主要落在哪些子调用里。读到的是这次采样窗口的数据。文章里的静态图片用于对照，点击放大和悬停读数需要打开生成的 HTML。

- **宽度看占比。** 方块越宽，表示经过这帧的调用栈占本轮采样结果的比例越大。以第 3.1 节为例，全图有 4,299 个 CPU 样本，其中 4,005 个经过 `OrderController.quote`，所以这一帧占全图宽度的约 93%。父帧的宽度已经包含上方的子调用。换成 `alloc`、`lock` 或 `wall` 时，要按对应事件和图中显示的单位来读。
- **高度看调用栈的深度。** 每个方块是一帧，方块本身等高。沿同一条分支从下往上看，下一层是调用者，上一层是被调用的方法。叠得越高，表示这条调用栈的层级越多。判断开销大小时看宽度。
- **横向位置用于排列调用栈。** 左右相邻的分支没有时间先后关系。

真实 Web 应用的全图里，通常还能看到日志、定时任务、后台线程池和连接池。先留一张全图，再按线程名和调用链区分来源。同步日志在请求线程里执行，异步日志也可能让请求卡在入队上，这两种情况都要沿请求栈查。独立运行的后台任务可以单独分析。需要聚焦业务包时，再用 2.8 节的 `--include` 导出一张对照图，比较占比时记得过滤后的样本总量已经变了。

### 3.1 如何使用火焰图分析 CPU 热点

CPU 持续偏高时，用 `cpu` 火焰图找哪段代码占用了最多的 CPU 样本，再判断热点落在方法本身还是它调用的其他方法里。

**示例：**

```bash
profiler start --event cpu
# 采样几秒后停止，导出 HTML 火焰图
profiler stop --format html --file /tmp/cpu.html
```

![CPU 火焰图](https://raw.githubusercontent.com/duwei0227/picbed/fa88f9b522dc47f3e7fed6c56540268efb7c0f91/arthas/profiler/cpu.svg)

1. **先看全图有多少样本。** 最底下的 `all` 有 **4,299** 个样本。沿最宽的分支往上看，能看到 Tomcat 和 Spring MVC 的调用帧，说明这条热点路径来自处理 Web 请求的线程。
2. **沿宽分支找到业务方法。** 继续往上找到 `OrderController.quote`，这一帧占了 **4,005** 个样本，约 **93.16%**。这是它在本轮 CPU 采样中的占比。
3. **看这一帧上面还有没有子调用。** 图里的 `quote` 已经是这条分支的栈顶。回到方法里，可以看到它在循环做整数运算。这一轮应先排查的就是 `quote` 自己的计算逻辑。
4. **把请求里的 CPU 开销和后台开销分开看。** 如果宽分支通向日志格式化、JSON 序列化或编码，先往下看它是不是从请求方法调过去的。在同一条请求栈里，就把它算在请求开销里。若宽分支来自日志消费线程或定时任务线程池，单独记下它的占比。后台线程也会占用进程 CPU，要结合全图判断它是不是挤占了请求线程的 CPU。

### 3.2 如何使用火焰图分析内存分配

GC 变频繁、怀疑对象创建得太多时，用 `alloc` 火焰图找分配集中在哪条调用链，再查具体在创建什么对象。

**示例：**

```bash
profiler start --event alloc
# 采样几秒后停止，导出 HTML 火焰图
profiler stop --format html --file /tmp/alloc.html
```

![分配火焰图](https://raw.githubusercontent.com/duwei0227/picbed/fa88f9b522dc47f3e7fed6c56540268efb7c0f91/arthas/profiler/alloc.svg)

1. **找覆盖分配样本最多的业务帧。** 全图有 **31,042** 个样本，沿最宽的分支往上找到 `OrderController.export`，它覆盖 **30,558** 个样本，约 **98.44%**。
2. **往上看分配发生在哪个子调用。** `export` 上方有两个分支。直接创建 `byte[][]` 占 **6** 个样本，`Arrays.setAll → lambda → byte[]` 占 **30,552** 个。先看后面这条宽分支。
3. **对照代码确认在分配什么。** lambda 每次创建一个 32 KiB 的字节数组，一轮会创建 64 个，光数组内容就有 2 MiB。这就定位到了要优先排查的那批数组。图上的 **30,552** 是样本计数，想知道这些对象是否长期存活，还需要另外检查堆。
4. **同一种对象要看它从哪条栈分配出来。** `byte[]` 也可能来自日志编码、HTTP 响应序列化或后台批处理。遇到几处分支都以 `byte[]` 收尾，就往下看到第一个能区分来源的方法。如果日志编码在请求栈里，它也是这次请求触发的分配。独立后台任务的分配可以另算，同时保留在全图里评估它对 GC 的影响。

### 3.3 如何使用火焰图分析锁竞争

并发升高后吞吐量上不去，怀疑线程在等锁时，用 `lock` 火焰图找获取锁时发生竞争的代码位置。

**示例：**

```bash
profiler start --event lock
# 采样几秒后停止，导出 HTML 火焰图
profiler stop --format html --file /tmp/lock.html
```

![锁竞争火焰图](https://raw.githubusercontent.com/duwei0227/picbed/fa88f9b522dc47f3e7fed6c56540268efb7c0f91/arthas/profiler/lock.svg)

1. **从锁对象帧往下找业务方法。** 图上两条分支都以 `java.lang.Object` 结尾，往下一层都能找到 `OrderController.reserve`。这两条分支分别占 **24** 和 **217** 个样本，合计 **241**，正好是全图的样本数。
2. **回代码确认是哪把锁。** `java.lang.Object` 只给出了对象类型。打开 `reserve`，里面只有一处 `synchronized (inventory)`，并发线程进入这里都要争用同一个对象。
3. **看拿到锁之后做了什么。** 线程拿到锁后还会在同步块里停约 **20 ms**，这段时间其他线程进不来。排查就落到了 `inventory` 锁的临界区，先看这段等待能不能移到同步块外面。
4. **区分请求正在等的锁和后台线程自己的锁。** 日志 appender、连接池和线程池也可能有竞争。先沿锁帧往下看，确定等锁的是请求线程还是独立的后台线程。请求线程如果卡在日志 appender 或连接池，继续查那条链。只有后台线程出现在某个竞争栈上时，可以先把它单独记下来。还想查谁持有锁，再用线程转储对照。

### 3.4 如何使用火焰图分析线程等待

请求慢、CPU 却不高时，用 `wall` 火焰图找处理请求的线程在哪条调用栈上停留，再判断它在等什么。

**示例：**

```bash
profiler start --event wall
# 采样几秒后停止，导出 HTML 火焰图
profiler stop --format html --file /tmp/wall.html
```

![Wall 火焰图](https://raw.githubusercontent.com/duwei0227/picbed/fa88f9b522dc47f3e7fed6c56540268efb7c0f91/arthas/profiler/wall.svg)

1. **先找处理请求的分支。** 全图有 **33,219** 个样本，里面还包括后台线程和空闲线程。定位业务等待时，沿请求处理栈找到 `OrderController.delivery`。这一帧覆盖 **2,851** 个样本，约占全图 **8.58%**。
2. **在这条分支里继续往上看。** `delivery` 的 **2,851** 个样本都经过 `LockSupport.parkNanos → Unsafe.park`。全图占比只有 8.58%，但从这条业务分支看，采到的样本全部落在等待路径上。
3. **回代码确认等待来源。** `delivery` 里用 `parkNanos` 模拟约 **75 ms** 的下游等待。这张图把排查范围缩小到了这段等待。要知道单次请求实际等了多久，再看请求耗时记录。
4. **先看 `park` 底下是谁在等。** 如果下面是线程池工作线程在队列里取任务，而且栈上还没有请求处理帧，通常是在空闲等活。日志消费线程等队列里的下一条消息，也可以先放到一边。若同样的 `park` 出现在 Controller 调用链上，例如请求正在等连接、等异步任务完成，或者等日志入队，就要继续查这段等待。


