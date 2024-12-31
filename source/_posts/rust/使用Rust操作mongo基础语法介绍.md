---
layout: post
title: 使用Rust操作mongo基础语法介绍
categories: [Rust]
tags: mongo
permalink: rust/mongo.html

---



### 一、添加依赖

```toml
futures = "0.3.31"
mongodb = "3.1.1"
serde = {version = "1.0.217", features = ["derive"]}
tokio = {version = "1.42.0", features = ["full"]}
```



* `futures`: 结果集中使用`try_next`获取数据时使用
* `mongodb`:连接`mongo`数据库驱动包
* `serde`: 用于将文档和`struct`进行转换，提供序列号和反序列化的能力
* `tokio`:提供异步运行时环境



### 二、基础语法

#### 1、配置连接client

```rust
let uri = "连接字符串";

let mut client_options = ClientOptions::parse(uri).await?;
client_options.max_pool_size = Some(50);  // 设置最大连接数
client_options.min_pool_size = Some(50);
client_options.connect_timeout = Some(Duration::from_secs(1));  // 连接超时时间

let client = Client::with_options(client_options)?;
```



完整的连接选项文档：[https://www.mongodb.com/zh-cn/docs/drivers/rust/current/fundamentals/connections/connection-options/](https://www.mongodb.com/zh-cn/docs/drivers/rust/current/fundamentals/connections/connection-options/)



#### 2、获取数据库和集合

```rust
let database = client.database("test");
let instance_coll: Collection<Document> = database.collection("instance");
```



#### 3、数据查询

获取数据集`collection`

```rust
let instance_coll: Collection<Document> = database.collection("instance");
```

此处集合数据类型可以使用 `Document`或者自定义`Struct`



使用 `find_one`查询一条记录

```rust
let camunda_instance_id = "2ca88c71-3636-11ec-b356-26c4328fcfef";
let instance =instance_coll.find_one(doc! {
    "instance_id": camunda_instance_id
}).await?;
```



使用`find`查询多条记录，返回集合

```rust

let mut cursor = instance_coll.find(doc! {
    "apply": "duw17"
}).await?;

while let Some(doc) = cursor.try_next().await? {
    println!("{:#?}", doc);
}
```



**`try_next`需要添加`futures crate`后，引入 `use futures::TryStreamExt;`**



#### 4、数据插入

使用 `insert_one`插入单个文档

```rust
// 使用doc!不指定具体数据类型
let insert_doc = doc! {
    "id": 222222222222222222i64,
    "instance_id": "222222222222222222",
    "example_status": 1,
    "apply": "duw17",
    "template_code": "314837782482464786",
};

// 自定义数据struct
// let insert_struct = Instance {
//     id: 11111111111111111u64,
//     instance_id: "11111111111111111".to_string(),
//     example_status: 1,
//     apply: "duw17".to_string(),
//     template_code: "314837782482464786".to_string(),
// };

let res = instance_coll.insert_one(insert_doc).await?;
println!("Inserted a document with _id: {}", res.inserted_id);
```



使用 `insert_many` 插入多个文档

```rust
// 此处同理可以使用具体的 struct
let insert_docs = vec![
    doc! {
        "id": 33333333333333333i64,
        "instance_id": "33333333333333333",
        "example_status": 0,
        "apply": "liuy2905",
        "template_code": "314837782482464786",
    },
    doc! {
        "id": 4444444444444444444i64,
        "instance_id": "4444444444444444444",
        "example_status": 1,
        "apply": "duw17",
        "template_code": "314837782482464786",
    },
];

let insert_many_result = instance_coll.insert_many(insert_docs).await?;
println!("Inserted documents with _ids:");
for (_key, value) in &insert_many_result.inserted_ids {
    println!("{}", value);
}
```



### 三、示例

```rust
use std::time::Duration;

use mongodb::{
    bson::{doc, Document}, options::ClientOptions, Client, Collection
};
use futures::TryStreamExt;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct Instance {
    id: u64,
    instance_id: String,
    example_status: u8,
    apply: String,
    template_code: String,
}

#[tokio::main]
async fn main() -> mongodb::error::Result<()> {
    let uri = "mongodb://mongo_testopr:Bpp_2023@10.0.195.200:22001,10.0.195.201:22001,10.0.195.202:22001/bpp_test?authSource=admin&replicaSet=rs1&readPreference=secondaryPreferred";

    let mut client_options = ClientOptions::parse(uri).await?;
    client_options.max_pool_size = Some(50);
    client_options.min_pool_size = Some(50);
    client_options.connect_timeout = Some(Duration::from_secs(1));
    
    let client = Client::with_options(client_options)?;

    let database = client.database("bpp_test");
    let instance_coll: Collection<Document> = database.collection("instance");

    // let instance_coll_count = instance_coll.count_documents(doc! {}).await?;
    // println!("instance_coll_count: {}", instance_coll_count);

    // let instance_id: u64 = 240873362090934272;
    // let camunda_instance_id = "2ca88c71-3636-11ec-b356-26c4328fcfef";
    // let instance =instance_coll.find_one(doc! {
    //     "instance_id": camunda_instance_id
    // }).await?;

    // println!("Found a instance:\n{:#?}", instance);

    // let mut cursor = instance_coll.find(doc! {
    //     "apply": "duw17"
    // }).await?;

    // while let Some(doc) = cursor.try_next().await? {
    //     println!("{:#?}", doc);
    // }

    let insert_doc = doc! {
        "id": 222222222222222222i64,
        "instance_id": "222222222222222222",
        "example_status": 1,
        "apply": "duw17",
        "template_code": "314837782482464786",
    };

    // let insert_struct = Instance {
    //     id: 11111111111111111u64,
    //     instance_id: "11111111111111111".to_string(),
    //     example_status: 1,
    //     apply: "duw17".to_string(),
    //     template_code: "314837782482464786".to_string(),
    // };

    // let res = instance_coll.insert_one(insert_doc).await?;
    // println!("Inserted a document with _id: {}", res.inserted_id);

    let insert_docs = vec![
        doc! {
            "id": 33333333333333333i64,
            "instance_id": "33333333333333333",
            "example_status": 0,
            "apply": "liuy2905",
            "template_code": "314837782482464786",
        },
        doc! {
            "id": 4444444444444444444i64,
            "instance_id": "4444444444444444444",
            "example_status": 1,
            "apply": "duw17",
            "template_code": "314837782482464786",
        },
    ];

    let insert_many_result = instance_coll.insert_many(insert_docs).await?;
    println!("Inserted documents with _ids:");
    for (_key, value) in &insert_many_result.inserted_ids {
        println!("{}", value);
    }

    Ok(())
}
                
```



### 四、参考文档

[https://www.mongodb.com/zh-cn/docs/drivers/rust/current/quick-start/](https://www.mongodb.com/zh-cn/docs/drivers/rust/current/quick-start/)

