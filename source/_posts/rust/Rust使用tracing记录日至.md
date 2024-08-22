---
layout: post
title: Rust使用tracing记录日志
categories: [Rust]
tags: serde
permalink: rust/tracing.html
---



### 1、添加依赖

```
tracing = "0.1.40"
tracing-appender = "0.2.3"
tracing-subscriber = { version = "0.3.18", features = ["json"] }    # 需要输出格式为json时，需要启用json feature
```



### 2、使用

#### 2.1 自定义RollingFile

```rus
let file_appender = RollingFileAppender::builder()
        .rotation(Rotation::DAILY)
        .filename_prefix("rust")
        .filename_suffix(".log")
        .max_log_files(5)
        .build("logs")
        .expect("failed to initialize rolling file appender");
```



#### 2.2 自定义format

```rus
let format = fmt::format()
        .with_target(false)
        .with_line_number(true)
        .with_thread_names(true)
        .compact();
```



#### 2.3 函数参数

在输出日志时，同步输出现场参数信息，可以通过在函数增加宏引用 `#[tracing::instrument]`

```rust
#[tracing::instrument]
fn say_number(number: usize) {
    info!("number is {}", number);
}
```



### 3、示例

```rust
use tracing::{info, span, Level};
use tracing_appender::rolling::{RollingFileAppender, Rotation};
use tracing_subscriber::fmt::writer::MakeWriterExt;
use tracing_subscriber::fmt;


#[tracing::instrument]
fn say_number(number: usize) {
    info!("number is {}", number);

}

fn main() {
    let file_appender = RollingFileAppender::builder()
        .rotation(Rotation::DAILY)
        .filename_prefix("rust")
        .filename_suffix(".log")
        .max_log_files(5)
        .build("logs")
        .expect("failed to initialize rolling file appender");

    let (file, _guard) = tracing_appender::non_blocking(file_appender);
    let (stdout, _guard) = tracing_appender::non_blocking(std::io::stdout());

    let format = fmt::format()
        .with_target(false)
        .with_line_number(true)
        .with_thread_names(true)
        .compact();

    tracing_subscriber::fmt()
        .with_writer(stdout.and(file))
        .event_format(format)
        .init();
    info!("hello world");
    info!("hello rust");
    info!("hello 中国");

    // 携带额外的信息在输出日志信息中
    let _span = span!(Level::INFO, "say", nums=4).entered();

    for i in 1..=4 {
        say_number(i);
    }
}
```

