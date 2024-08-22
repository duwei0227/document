---
layout: post
title: Rust使用log4rs进行日志输出管理
categories: [Rust]
tags: log
permalink: rust/log4rs.html
---


#### 一、依赖引入

```shell
cargo add log  -- 类比于Java中的slf4j，一个日志切面
```

日志输出实际的框架：

```shell
cargo add log4rs
```

常见的需要启用的`feature`特性

```
["console_appender", "rolling_file_appender", "size_trigger", "time_trigger", "pattern_encoder", "gzip"] 
```



* `console_appender` 日志输出到控制台
* `rolling_file_appender` 日志输出到文件，文件支持滚动策略
* `size_trigger` 按照文件大小实现滚动
* `time_trigger` 按照日期实现滚动
* `pattern_encoder` 日志内容输出格式
* `gzip` 压缩特性



#### 二、Pattern

示例：

```
{d(%Y-%m-%dT%H:%M:%S)} {h({l}):<5.5} {M} {f} {L} ---> {m}{n}
```

* `d` 日期，当前时间，默认情况按照`ISO 8601`格式输出，时区默认区系统，可以指定UTC
  * `Y`：年
  * `m`：月
  * `d`：天
  * `H`：时
  * `M`：分
  * `S`：秒
* `h`：内容高亮` red:error warning blue:info `
* `l`：`level` 日志级别 `trace` `debug` `info` `warn` `error`
* `M`：模块名，日志消息来源模块，`???`未提供
* `f`：`file`，日志消息来源文件，`???`未提供
* `L`：`line`，日志消息来源行，`???`未提供
* `m`：`message` 消息
* `n`：换行

完整的`Pattern`配置可以查看如下文档

https://docs.rs/log4rs/latest/log4rs/encode/pattern/index.html#formatters



#### 三、Appender

日志文件输出位置，支持控制台`Console`以及文件



##### 1、console

```yaml
console_name:
	kind: console
	target: stdout
	encoder:
		pattern: xxx
```



* `kind`：指定日志输出目标为 `console`，需要开启`console_appender`特征
* `target`：支持 `stdout` 和 `stderr`，默认为 `stdout`
* `encoder`：输出格式



##### 2、Rolling file

```yaml
rolling_file_name:
    kind: rolling_file
    path: "log/requests.log"
    encoder:
      pattern: "{d(%Y-%m-%dT%H:%M:%S)} {h({l}):<5.5} {M} {f} {L} ---> {m}{n}"
    policy:
      trigger:
          # 按照时间滚动
          kind: time
          interval: 1 second # second minute hour day week month year
          # 按照大小限制
          # kind: size
          # limit: 1mb  # b kb mb gb tb
      roller:
          kind: fixed_window
          pattern: "log/old-rolling_file-{}.gz"  # 重命名文件,必须包含 {}，用于索引占位 如果文件名后缀以 .gz 结尾并启用 gzip 特性，归档文件会被压缩
          base: 0
          count: 2  # 文件总数
```



* `kind`：指定日志输出目标为`rolling file`，需要开启`rolling_file_appender`特征
* `path`：指定当前活动日志文件路径，文件名称
* `encoder`：指定输出消息格式
* `policy`：滚动策略
* `trigger`：触发器
  * `kind`：类型，支持 `time` 和 `size`
  * `interval`：当`kind`为`time`时生效，时间周期，支持 `second minute hour day week month year`
  * `limit`：当`kind`为`size`时生效，按照文件大小上限滚动，支持 `b kb mb gb tb`
* `roller`：日志归档滚动策略
  * `kind`：支持 `fixed_window`固定文件数量和`delete`，`delete`时不需要指定其他格式
  * `pattern`：日志文件归档转储时文件名格式，其中 `{}`为必须用于指定文件索引数，如果文件名以`.gz`结尾且启用 `gzip`特征，转储时会对文件进行压缩
  * `base`：归档文件开始索引
  * `count`：归档文件总数，超过上限后，会不断滚动覆盖索引为最小的文件



参考：

https://github.com/estk/log4rs/blob/main/docs/Configuration.md#appender-config



#### 四、Logger

用于指定不同模块日志级别以及日志输出目标

```yaml
root:
  level: info
  appenders:
    - stdout
    - requests
```



* `root`：固定，默认处理方式
* `level`：日志级别
* `appenders`：输出目标，支持配置多个



```
loggers:
  app::requests:
    level: info
    appenders:
      - requests
    additive: false
```

* `loggers`：对于特定路径日志级别或者输出目标自定义
* `app::requests`：路径
* `additive`：`true`或`false` 是否将日至消息追加到上级 `appenders`中



#### 五、示例

```rust
log4rs::init_file("log4rs.yml", Default::default()).unwrap();
```



```yam
# Scan this file for changes every 30 seconds
refresh_rate: 30 seconds

appenders:
  # An appender named "stdout" that writes to stdout
  stdout:
    kind: console
    encoder:
      pattern: "{d(%Y-%m-%dT%H:%M:%S)} {h({l}):<5.5} {M} {f} {L} ---> {m}{n}"

  # An appender named "requests" that writes to a file with a custom pattern encoder
  requests:
    kind: rolling_file
    path: "log/requests.log"
    encoder:
      pattern: "{d(%Y-%m-%dT%H:%M:%S)} {h({l}):<5.5} {M} {f} {L} ---> {m}{n}"
    policy:
      trigger:
          # 按照时间滚动
          # kind: time
          # interval: 1 second # second minute hour day week month year
          # 按照大小限制
          kind: size
          limit: 1kb  # b kb mb gb tb
      roller:
          kind: fixed_window
          pattern: "log/old-rolling_file-{}.gz"  # 重命名文件,必须包含 {}，用于索引占位 如果文件名后缀以 .gz 结尾并启用 gzip 特性，归档文件会被压缩
          base: 0
          count: 5  # 文件总数

# Set the default logging level to "warn" and attach the "stdout" appender to the root
root:
  level: info
  appenders:
    - stdout
    - requests

loggers:
  # Raise the maximum log level for events sent to the "app::backend::db" logger to "info"
  app::backend::db:
    level: info

  # Route log events sent to the "app::requests" logger to the "requests" appender,
  # and *not* the normal appenders installed at the root
  app::requests:
    level: info
    appenders:
      - requests
    additive: false
```

