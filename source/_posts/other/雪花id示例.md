---
layout: post
title: 雪花id生成示例
categories: [其他]
permalink: other/snowflake.html
---

### 

Twitter Snowflake ID 是一个 64 位的整数，结构如下：

| 位数 | 含义             | 描述                                                         |
| ---- | ---------------- | ------------------------------------------------------------ |
| 1    | 符号位           | 总是 0，表示正数                                             |
| 41   | 时间戳（毫秒级） | 相对于某个时间纪元的时间差                                   |
| 10   | 机器标识         | 通常由数据中心 ID 和机器 ID 组成，保证分布式系统中机器唯一性 |
| 12   | 序列号           | 每毫秒内生成的序列号，用于避免同一毫秒内的 ID 冲突           |



**实现步骤**

1. 定义一个固定的时间纪元（epoch）。
2. 使用高精度时间戳获取当前时间，计算与 epoch 的差值作为时间部分。
3. 将机器 ID 和序列号填充到对应位上。
4. 按照位移规则生成最终的 64 位整数。



重点是对于时间戳的比较：

* 时间戳相同：需要考虑序号生产的唯一性，如果超过最大序列需要等待到下一毫秒
* 当前时间大于上一时间：从0开始直接计数
* 当前时间小于上一时间：可能发生了时钟回拨，可以报错或者按照上一时间继续处理生成



另外需要根据每一块数据区域位数计算允许的最大上限值，例如工作中心允许的位数为5，允许的最大值：`(1 << 5 ) - 1`



### 一、Rust语言

```rust
use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::Arc;
use std::thread;
use std::thread::JoinHandle;
use std::time::SystemTime;

/// 位数	含义	描述
/// 1	符号位	总是 0，表示正数
/// 41	时间戳（毫秒级）	相对于某个时间纪元的时间差
/// 10	机器标识	通常由数据中心 ID 和机器 ID 组成，保证分布式系统中机器唯一性
/// 12	序列号	每毫秒内生成的序列号，用于避免同一毫秒内的 ID 冲突

const EPOCH: u64 = 1735689600000; // 自定义纪元（2025-01-01 00:00:00 UTC）
const DATA_CENTER_ID_BITS: u64 = 5; // 数据中心 ID 位数
const MACHINE_ID_BITS: u64 = 5; // 机器 ID 位数
const SEQUENCE_BITS: u64 = 12; // 序列号位数

const MAX_DATA_CENTER_ID: u64 = (1 << DATA_CENTER_ID_BITS) - 1;
const MAX_MACHINE_ID: u64 = (1 << MACHINE_ID_BITS) - 1;
const MAX_SEQUENCE: u64 = (1 << SEQUENCE_BITS) - 1;

const MACHINE_ID_SHIFT: u64 = SEQUENCE_BITS;
const DATA_CENTER_ID_SHIFT: u64 = SEQUENCE_BITS + MACHINE_ID_BITS;
const TIMESTAMP_SHIFT: u64 = SEQUENCE_BITS + MACHINE_ID_BITS + DATA_CENTER_ID_BITS;

struct Snowflake {
    last_timestamp: AtomicU64,
    data_center_id: u64,
    machine_id: u64,
    sequence: AtomicU64,
}

impl Snowflake {
    pub fn new(data_center_id: u64, machine_id: u64) -> Snowflake {
        if data_center_id > MAX_DATA_CENTER_ID {
            panic!("Data center ID exceeds the maximum value!");
        }
        if machine_id > MAX_MACHINE_ID {
            panic!("Machine ID exceeds the maximum value!");
        }

        Snowflake {
            last_timestamp: AtomicU64::new(0),
            data_center_id,
            machine_id,
            sequence: AtomicU64::new(0),
        }
    }

    pub fn next_id(&self) -> u64 {
        let mut timestamp = current_millis();

        // 获取并更新最后的时间戳
        let last_timestamp = self.last_timestamp.load(Ordering::Relaxed);

        if timestamp < last_timestamp {
            // 处理时钟会退
            eprintln!(
                "Clock moved backwards. Refusing to generate ID for {} milliseconds.",
                last_timestamp - timestamp
            );
            timestamp = last_timestamp;
        }

        if timestamp == last_timestamp {
            // 同一毫秒内增加序列号
            // 确保序号在一个时间单位内循环使用，避免超出位数限制，保证 ID 格式的正确性
            let seq = self.sequence.fetch_add(1, Ordering::Relaxed) & MAX_SEQUENCE;
            if seq == 0 {
                // 序列号耗尽，等待下一毫秒
                timestamp = wait_next_millis(last_timestamp);
                self.sequence.store(0, Ordering::Relaxed);
            }
        } else {
            // 不同毫秒内重置序列号
            self.sequence.store(0, Ordering::Relaxed);
        }

        // 更新最后的时间戳
        self.last_timestamp.store(timestamp, Ordering::Relaxed);

        // 生成雪花 ID
        ((timestamp - EPOCH) << TIMESTAMP_SHIFT)
            | (self.data_center_id << DATA_CENTER_ID_SHIFT)
            | (self.machine_id << MACHINE_ID_SHIFT)
            | self.sequence.load(Ordering::Relaxed)
    }
}

fn current_millis() -> u64 {
    SystemTime::now()
        .duration_since(SystemTime::UNIX_EPOCH)
        .unwrap()
        .as_millis() as u64
}

fn wait_next_millis(last_timestamp: u64) -> u64 {
    let mut timestamp = current_millis();
    while timestamp <= last_timestamp {
        timestamp = current_millis();
    }
    timestamp
}

fn main() {
    let snowflake = Arc::new(Snowflake::new(1, 1));

    let handlers: Vec<_> = (0..10)
        .map(|_| {
            let snowflake = snowflake.clone();
            thread::spawn(move || {
                for _ in 0..10 {
                    let id = snowflake.next_id();
                    println!("Generate ID: {}", id);
                }
            })
        })
        .collect();
    for h in handlers {
        h.join().unwrap();
    }
}

```

