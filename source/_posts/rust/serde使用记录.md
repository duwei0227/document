---
layout: post
title: Rust使用serde进行序列化
categories: [Rust]
tags: serde
permalink: rust/serde.html
---



### 1、添加依赖

```
cargo add serde --features=derive
cargo add serde_json
```



### 2、常用属性

#### 2.1 全局修改属性名称格式

* 位置：增加在 `struct`位置

* 配置：`#[serde(rename_all = "...")]`，支持 `"lowercase"`, `"UPPERCASE"`, `"PascalCase"`, `"camelCase"`, `"snake_case"`, `"SCREAMING_SNAKE_CASE"`, `"kebab-case"`, `"SCREAMING-KEBAB-CASE"`

* 属性名称需要以 下划线(`_`)分割

   

  示例：

  ```rust
  #[derive(Serialize, Deserialize, Debug)]
  #[serde(rename_all="camelCase")]  // json字段格式为 camelCase 例如 stringName
  pub struct StructWithCustomDate {
      pub user_name: String,
  }
  ```

  

  序列化时要求`json`中字段名为 `userName`



#### 2.2 默认值 `#[serde(default)]`

使用属性类型默认值，即由 `Default Trait`提供的默认值，例如：`i32 默认为0`

需要在具体属性上标注

示例：

```rust
// 默认值
#[serde(default)]
pub age: i32,
```



#### 2.3 默认值，指定函数 `#[serde(default = "path")]` 

由指定的函数提供属性缺省值。

需要在具体属性上标注

示例：

```rust
// 自定义默认值处理函数
#[serde(default="my_sex")]
pub sex: String,

fn my_sex() -> String {
    String::from("Female")
}
```



#### 2.4 属性跳过 `#[serde(skip)]` 

可以通过 `#[serde(skip_serializing)]` 和 `#[serde(skip_deserializing)]`  单独针对序列化或反序列化过程



在反序列化的时候，`serde`会使用 `Default::default()` 或 `default = "..."` 提供属性默认值

```rust
#[serde(skip)]
bidder: String,
```



#### 2.5 外部module自定义序列化过程 `#[serde(with = "module")]`

```rust
mod my_who_care {
    use serde::{self, Deserialize, Serializer, Deserializer};

    pub fn serialize<S>(属性名: 属性类型, serializer: S) -> Result<S::Ok, S::Error>
        where S: Serializer, {
            // 序列化逻辑
        serializer.serialize_str()
    }

    pub fn deserialize<'de, D>(deserializer: D,) -> Result<String, D::Error>
    where D: Deserializer<'de, > {
        let s = String::deserialize(deserializer)?;
        // 反序列化逻辑
        Ok()
    }
}
```



### 3、自定义日期序列化

```rust
use chrono::{DateTime, Utc};
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(rename_all="camelCase")]  // json字段格式为 camelCase 例如 stringName
pub struct StructWithCustomDate {
    // DateTime supports Serde out of the box, but uses RFC3339 format. Provide
    // some custom logic to make it use our desired format.
    #[serde(with = "my_date_format")]
    pub time_stamp: DateTime<Utc>,
}

mod my_date_format {
    use chrono::{DateTime, Utc, NaiveDateTime};
    use serde::{self, Deserialize, Serializer, Deserializer};

    const FORMAT: &'static str = "%Y-%m-%d %H:%M:%S";

    // The signature of a serialize_with function must follow the pattern:
    //
    //    fn serialize<S>(&T, S) -> Result<S::Ok, S::Error>
    //    where
    //        S: Serializer
    //
    // although it may also be generic over the input types T.
    pub fn serialize<S>(
        date: &DateTime<Utc>,
        serializer: S,
    ) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let s = format!("{}", date.format(FORMAT));
        serializer.serialize_str(&s)
    }

    // The signature of a deserialize_with function must follow the pattern:
    //
    //    fn deserialize<'de, D>(D) -> Result<T, D::Error>
    //    where
    //        D: Deserializer<'de>
    //
    // although it may also be generic over the output types T.
    pub fn deserialize<'de, D>(
        deserializer: D,
    ) -> Result<DateTime<Utc>, D::Error>
    where
        D: Deserializer<'de>,
    {
        let s = String::deserialize(deserializer)?;
        let dt = NaiveDateTime::parse_from_str(&s, FORMAT).map_err(serde::de::Error::custom)?;
        Ok(DateTime::<Utc>::from_naive_utc_and_offset(dt, Utc))
    }
}

fn main() {
    let json_str = r#"
      {
        "timeStamp": "2017-02-16 21:54:30",
        "bidder": "Skrillex",
        "whoCare": "probiecoder"
      }
    "#;

    let data: StructWithCustomDate = serde_json::from_str(json_str).unwrap();
    println!("{:#?}", data);

    let serialized = serde_json::to_string_pretty(&data).unwrap();
    println!("{}", serialized);
}
```



输出结果：

```
StructWithCustomDate {
    time_stamp: 2017-02-16T21:54:30Z,
}
{
  "timeStamp": "2017-02-16 21:54:30"
}
```



更多内容请查阅`serde`官网内容：[https://serde.rs/](https://serde.rs/)

