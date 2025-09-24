---
layout: post
title: Eclipse Memory Analyzer分析堆文件技巧
categories: [Java]
permalink: java/mat_usage.html
tags: MAT

---





### 一、巧用OBJECTS获取Map对象中的key和每个key使用的retainedHeapSize

`OBJECTS`是一个关键字，用来“扁平化”（`flatten`）一个对象数组 / 引用数组 /对象集合，使得你可以把“一个数组 / 列表 /子查询返回的集合”中的每一个元素当作单独一行来处理



举例：

现在想要获取`Camunda`的`instanceCache`中存储了哪些`key`，且每一个`key`的`retainedHeapSize`应该如何获取？

![image-20250924165742046](https://raw.githubusercontent.com/duwei0227/picbed/main/document/image-20250924165742046.png)



#### 1、选中cache行执行OQL

```
SELECT * FROM OBJECTS 1529719
```

![image-20250924170004006](https://raw.githubusercontent.com/duwei0227/picbed/main/document/image-20250924170004006.png)



从执行结果来说，返回了`ConcurrentHashMap`实例本身以及所保留的内存



#### 2、调整select结果，获取map中的元素项

在获取`map`实例中具体的元素项的时候，就需要使用`OBJECTS`进行扁平化展开

```
SELECT objects o.table.@referenceArray FROM OBJECTS 1529719 o 
```

* 通过`.`获取对象属性，此处`table`为`ConcurrentHashMap`的一个属性
* `@referenceArray` 是一个用于访问 **对象引用数组** 的内建属性



![image-20250924170214350](https://raw.githubusercontent.com/duwei0227/picbed/main/document/image-20250924170214350.png)



此时已经获取到`map`实例中存储的每个`key`以及所保留的内存，但是由于`key`本身还是存储在`Node`中无法直观查看，需要进一步调整语句获取`Node`对象中的具体属性



#### 3、使用嵌套查询获取`Node`对象中的属性

嵌套查询结果也是一个列表，需要使用`OBJECTS`平铺展开

```
SELECT toString(s.key) AS key, s.@retainedHeapSize FROM OBJECTS ( SELECT OBJECTS o.table.@referenceArray FROM OBJECTS 1529719 o  ) s 
```

* `@retainedHeapSize` 是一个 **内建属性（Bean 属性 / OQL attribute）**，用来表示某个对象的 **Retained Heap 大小**
* `toString`是用来将key的结果转为字符串输出
* `s.key`中的`key`是`Node`对象的属性，也可以用来获取`Node`中的`val`，`hash`等属性

![image-20250924171124031](https://raw.githubusercontent.com/duwei0227/picbed/main/document/image-20250924171124031.png)





**Shallow Heap Size vs Retained Heap Size**

- *Shallow Heap*（也叫 *usedHeapSize* / `@usedHeapSize`）是对象自身在内存中占用的大小，不包括它引用子对象的那部分。
- *Retained Heap*（`@retainedHeapSize`）则包括这个对象及其控制／支配（如果这个对象被回收时，子对象也可回收）的子对象占用的内存。
- 举例来说，如果对象 A 引用 B 和 C，且 B 和 C 没有被别的对象引用，那么 A 的 `@retainedHeapSize` = A 的 shallow + B 的 shallow + C 的 shallow



