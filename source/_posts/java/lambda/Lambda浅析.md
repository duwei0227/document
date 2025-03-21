---
layout: post
title: Lambda浅析
categories: [Java]

---



## 一、Lambda 表达式核心概念

### 1. 基本语法
```java
(parameters) -> expression
(parameters) -> { statements; }
```

### 2. 类型推断机制
```java
// 编译器自动推断参数类型
Function<String, Integer> lengthFunc = s -> s.length(); 
```

---



## 二、方法引用详解

### 1. 方法引用类型
| 类型                 | 语法格式                    | 等效 Lambda 表达式                        | 示例             |
| -------------------- | --------------------------- | ----------------------------------------- | ---------------- |
| 静态方法引用         | `ClassName::staticMethod`   | `args -> ClassName.staticMethod(args)`    | `Math::max`      |
| 实例方法引用（对象） | `object::instanceMethod`    | `args -> object.instanceMethod(args)`     | `list::add`      |
| 实例方法引用（类）   | `ClassName::instanceMethod` | `(obj, args) -> obj.instanceMethod(args)` | `String::length` |
| 构造方法引用         | `ClassName::new`            | `args -> new ClassName(args)`             | `ArrayList::new` |

### 2. 典型应用场景
```java
// 静态方法引用
List<Integer> numbers = Arrays.asList(1,2,3);
numbers.stream().map(String::valueOf); // 等效 s -> String.valueOf(s)

// 构造方法引用
Supplier<List<String>> listSupplier = ArrayList::new;

// 对象方法引用
Consumer<String> printer = System.out::println;
```

### 3. 方法引用 vs Lambda
```java
// Lambda 表达式
Function<String, Integer> f1 = s -> s.length();

// 方法引用
Function<String, Integer> f2 = String::length;
```

---



## 三、核心函数式接口

### 1. 四大基础接口
| 接口            | 方法签名            | 典型应用            |
| --------------- | ------------------- | ------------------- |
| `Supplier<T>`   | `T get()`           | 对象工厂/延迟初始化 |
| `Consumer<T>`   | `void accept(T t)`  | 集合遍历处理        |
| `Predicate<T>`  | `boolean test(T t)` | 数据过滤            |
| `Function<T,R>` | `R apply(T t)`      | 数据转换            |

### 2. 16 种原始类型
#### 输入/输出类型表
| 输入类型 \ 输出类型 | T                   | int                   | long                   | double                 |
| ------------------- | ------------------- | --------------------- | ---------------------- | ---------------------- |
| **T**               | `UnaryOperator<T>`  | `ToIntFunction<T>`    | `ToLongFunction<T>`    | `ToDoubleFunction<T>`  |
| **int**             | `IntFunction<R>`    | `IntUnaryOperator`    | `IntToLongFunction`    | `IntToDoubleFunction`  |
| **long**            | `LongFunction<R>`   | `LongToIntFunction`   | `LongUnaryOperator`    | `LongToDoubleFunction` |
| **double**          | `DoubleFunction<R>` | `DoubleToIntFunction` | `DoubleToLongFunction` | `DoubleUnaryOperator`  |

#### 典型接口详解
| 接口                  | 方法签名                 | 应用场景              | 示例代码                                    |
| --------------------- | ------------------------ | --------------------- | ------------------------------------------- |
| `IntFunction<R>`      | `R apply(int value)`     | int → 任意类型转换    | `IntFunction<String> f = i -> "No." + i;`   |
| `ToIntFunction<T>`    | `int applyAsInt(T t)`    | 对象 → int 提取       | `ToIntFunction<String> f = String::length;` |
| `IntUnaryOperator`    | `int applyAsInt(int)`    | int → int 运算        | `IntUnaryOperator add5 = x -> x + 5;`       |
| `DoubleToIntFunction` | `int applyAsInt(double)` | double → int 强制转换 | `DoubleToIntFunction f = d -> (int)d;`      |

### 3. 二元操作接口
| 接口                | 方法签名                 | 应用场景       |
| ------------------- | ------------------------ | -------------- |
| `BiFunction<T,U,R>` | `R apply(T t, U u)`      | 合并两个对象   |
| `BiConsumer<T,U>`   | `void accept(T t, U u)`  | 消费两个对象   |
| `BiPredicate<T,U>`  | `boolean test(T t, U u)` | 双参数条件判断 |

```java
// 合并两个列表
BiFunction<List<String>, List<String>, List<String>> merger = 
    (list1, list2) -> Stream.concat(list1.stream(), list2.stream())
                           .collect(Collectors.toList());
```

---



## 四、Lambda 高级特性

### 1. 变量捕获规则
```java
int base = 10;  // 必须为 effectively final
Function<Integer, Integer> adder = x -> x + base;
```

### 2. 闭包与内存管理
```java
Supplier<IntSupplier> createCounter() {
    int[] count = {0};  // 使用数组绕过 final 限制
    return () -> () -> count[0]++;
}
```

---



## 五、应用案例

### 1. 集合处理
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 过滤 + 转换 + 消费
names.stream()
     .filter(s -> s.length() > 3)         // Predicate
     .map(String::toUpperCase)            // Function
     .forEach(System.out::println);       // Consumer
```

### 2. 工厂模式
```java
Map<String, Supplier<Shape>> shapeFactory = new HashMap<>();
shapeFactory.put("circle", Circle::new);
shapeFactory.put("square", Square::new);

Shape shape = shapeFactory.get("circle").get();
```

---



## 六、性能优化

1. **避免自动装箱**：优先使用原始类型特化接口  
   ```java
   // 低效写法
   Function<Integer, Integer> f = i -> i * 2; 
   
   // 高效写法
   IntUnaryOperator f = i -> i * 2;
   ```

2. **缓存 Lambda 表达式**：对重复使用的表达式进行复用  
   ```java
   private static final Predicate<String> LENGTH_3 = s -> s.length() == 3;
   ```

3. **谨慎使用并行流**：仅在数据量大且无状态操作时启用  
   ```java
   list.parallelStream().map(...) 
   ```

---



## 附录：JDK 函数式接口全表

| 接口分类   | 核心接口        | 原始类型特化接口（示例）             | 二元接口            |
| ---------- | --------------- | ------------------------------------ | ------------------- |
| **生产型** | `Supplier<T>`   | `IntSupplier`, `DoubleSupplier`      | -                   |
| **消费型** | `Consumer<T>`   | `IntConsumer`, `DoubleConsumer`      | `BiConsumer<T,U>`   |
| **判断型** | `Predicate<T>`  | `IntPredicate`, `DoublePredicate`    | `BiPredicate<T,U>`  |
| **转换型** | `Function<T,R>` | `IntFunction<R>`, `ToIntFunction<T>` | `BiFunction<T,U,R>` |

---

