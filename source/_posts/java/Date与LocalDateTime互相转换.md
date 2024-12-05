---
layout: post
title: Date与LocalDateTime互相转换
categories: [Java]
permalink: java/date_localdatetime.html
---

### 



`Date`和`LocalDateTime`之间的互相转换需要借助`Instant`，`Instant`是时间线上的一个瞬时点，不包含时区信息



### 一、Date转LocalDateTime

转换顺序：

* `Date`转为`Instant`
* `Instant`转为`LocalDateTime`或者`LocalDate`

```java
Date date = new Date();                                                                

// 先转为Instant                                                                          
Instant instant = date.toInstant();                                                    
                                                                                       
LocalDateTime localDateTime = instant.atZone(ZoneId.systemDefault()).toLocalDateTime();
                                                                                       
LocalDateTime localDateTime = instant.atZone(ZoneId.ofOffset("UTC", ZoneOffset.ofHours
                                                                                       
LocalDateTime localDateTime = instant.atZone(ZoneId.ofOffset("UTC", ZoneOffset.ofHours
                                                                                       
LocalDateTime localDateTime = instant.atZone(ZoneOffset.of("+8")).toLocalDateTime();  
                                                                                       
LocalDateTime localDateTime = instant.atZone(ZoneOffset.of("-8")).toLocalDateTime();  
```



### 二、LocalDateTime转Date

转换顺序：

* `LocleDateTie`转为`Instant`
* `Instant`转为`Date`

```java
LocalDateTime now = LocalDateTime.now();     

// 此处的转换方式和Date转LocalDateTime相同
Instant instant = now.toInstant(ZoneOffset.of("+7"));          

Date date = Date.from(instant)
```





`Instant`转换的时候指定目标时区信息方式：

* `ZoneId.ofOffset("UTC", ZoneOffset.ofHours(5))` 基于`UTC`进行小时偏移
* `ZoneId.ofOffset("UTC", ZoneOffset.ofHoursMinutes(6, 5))`基于`UTC`进行小时和分钟偏移
* `ZoneId.systemDefault()`基于当前所在系统进行偏移
* `ZoneOffset.of("+8")`直接指定偏移，支持`+`和`-`时间前后偏移