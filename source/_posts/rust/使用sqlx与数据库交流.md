---
layout: post
title: 使用sqlx与数据库交流
categories: [Rust]
tags: sqlx
permalink: rust/sqlx.html

---



### 一、依赖添加

```toml
chrono = {version = "0.4.38", features = ["serde"]}
serde = { version = "1.0.210", features = ["derive"] }
serde_json = "1.0.132"
sqlx = { version = "0.8.2", features = ["mysql", "runtime-tokio", "chrono", "json", "tls-rustls"] }
tokio = { version = "1.40.0", features = ["full"]}
```

* `chrono`:处理日期类型字段，提供 `NaiveDateTime`和`DateTime<Utc>`,其中`NaiveDateTime`不包含时区信息，对应数据库的`DATETIME`类型，`DateTime<Utc>`包含`UTC`时区信息，对应数据库`TIMESTAMP`类型；`chrono`默认未实现`serde`的`Serialize`，需要启用`serde feature`支持。
* `serde`和`serde_json`:用于处理数据的序列化。
* `sqlx`: 用于与数据库交付的核心`crate`
* `tokio`: 异步运行`crate`



### 二、操作示例

#### 1、创建连接池

```rust
let pool = MySqlPoolOptions::new()
        .max_connections(5)
        .min_connections(2)
        .idle_timeout(Duration::from_secs(10))
        .acquire_timeout(Duration::from_secs(1))
        .connect("mysql://root:Root@localhost:3306/rust")
        .await?;
```



#### 2、数据插入

```rust
let insert_sql = r#"
    insert into student (name,age,birthday,create_date) values (?,?,?,?)"#;

for i in 0..10 {
    sqlx::query(&insert_sql)
        .bind(format!("probie-{}", i))
        .bind(10 + i)
        .bind(NaiveDateTime::parse_from_str(
            "1991-01-13 05:11:00",
            "%Y-%m-%d %H:%M:%S",
        )?)
        .bind(Utc::now())
        .execute(&pool)
        .await?;
}
```



* `bind`用于变量绑定替换，在`MySQL`中使用 `?` 作为变量占位符

* `execute`用于`sql`执行

* `NaiveDateTime::parse_from_str`用于将字符串格式日期转为 `NaiveDateTime`,常用的格式：

  `%Y`：四位年份

  `%m`：两位月份

  `%d`：两位日期

  `%H`：24 小时制的小时

  `%M`：分钟

  `%S`：秒



#### 3、数据查询并映射到 `serde_json.Value`

```rust
let select_sql = "select * from student";
let rows = sqlx::query(select_sql).fetch_all(&pool).await?;

// 将结果映射为serde_json::Value
let mut json_rows: Vec<Value> = Vec::new();

for row in rows {
    let mut json_row = HashMap::new();

    // 遍历所有列
    for (i, column) in row.columns().iter().enumerate() {
        let column_name = column.name();
        // 动态获取列的值并判断类型
        let value: Value = if let Ok(v) = row.try_get::<i64, _>(column_name) {
            json!(v)
        } else if let Ok(v) = row.try_get::<f64, _>(column_name) {
            json!(v)
        } else if let Ok(v) = row.try_get::<String, _>(column_name) {
            json!(v)
        } else if let Ok(v) = row.try_get::<bool, _>(column_name) {
            json!(v)
        } else if let Ok(v) = row.try_get::<NaiveDateTime, _>(column_name) {
            json!(v)
        } else if let Ok(v) = row.try_get::<DateTime<Utc>, _>(column_name) {
            json!(v)
        } else {
            Value::Null
        };

        json_row.insert(column_name.to_string(), value);
    }
    json_rows.push(json!(json_row));
}

// 打印 JSON 结果
println!("{}", serde_json::to_string_pretty(&json_rows)?);
```



#### 4、使用 query_as 将结果映射到struct

`query_as`函数运行时解析 SQL 语句，适合动态生成的查询，但没有编译时的类型安全检查



* `struct`需要通过`sqlx::FromRow`进行类型转换

  ```rust
  #[derive(Debug, FromRow)]
  struct Student {
      id: i64,
      name: String,
      age: i8,
      birthday: NaiveDateTime,
      create_date: Option<DateTime<Utc>>,
  }
  ```

  

* 使用 `sqlx::query_as::<struct, _>()`指定需要解析的类型

  ```rust
  // 将结果映射到 struct
  let select_one_sql = "select * from student where id = ?";
  let student: Student = sqlx::query_as::<_, Student>(select_one_sql)
      .bind(1) // 使用 bind 来绑定查询参数
      .fetch_one(&pool)
      .await?;
  ```

  

#### 5、使用query_as!宏将结果映射到struct

`query_as!`宏会在编译时检查 `SQL` 语句的正确性，要求 `SQL `是静态字符串，确保类型匹配，另外`query_as!`宏不要求`struct`通过`sqlx::FromRow`进行类型转换

```rust
let student = sqlx::query_as!(Student, "SELECT * FROM student WHERE id = ?", 1)
        .fetch_one(&pool)
        .await?;

    println!("{:?}", student);
```



错误提示：

* `expected string literal `:`SQL`语句不是直接的字符串，而是通过变量等形式引用，无法在编译期确定
* `error: set DATABASE_URL to use query macros online, or run cargo sqlx prepare to update the query cache`:在根目录下创建`.env`文件并添加`DATABASE_URL`环境变量可以解决问题



### 三、参考

https://docs.rs/sqlx/0.8.2/sqlx/index.html

https://crates.io/crates/sqlx#usage