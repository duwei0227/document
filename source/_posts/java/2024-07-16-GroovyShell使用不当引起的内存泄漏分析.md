---
layout: post
title: GroovyShell使用不当引起的内存泄漏分析
categories: [Java]
description: GroovyShell使用不当引起的内存泄漏分析
permalink: java/groovy-shell.html
tags: Groovy
---

#### 1、背景



#### 一、背景

项目中会使用`Groovy`动态计算字符串脚本，运行一段时间后监控发现内存使用占比较高。



#### 二、问题原因分析

##### 1、`dump`堆文件分析

```
jmap -dump:format=b,file=dump.hprof <pid>
```



##### 2、使用MAT分析堆文件

![memory leak](https://raw.githubusercontent.com/duwei0227/picbed/main/202407171538033.png)



分析堆文件可以查看到有一块区域内存使用较多，疑似存在泄漏问题，进一步查看  `Duplicate Classes`查看是否存在重复对象

![duplicate classes](https://raw.githubusercontent.com/duwei0227/picbed/main/202407171541093.png)



此时考虑`Groovy`执行存在内存泄漏的问题。



##### 3、代码分析

```java
var groovyShell = new GroovyShell();
Object obj = groovyShell.evaluate(scriptContent);
```

`Groovy`在执行脚本前会见字符串内容通过反射的形式转为`Class`文件，每次都会通过反射创建一个 `Script` 实例对象

```java
public static Script newScript(Class<? extends Script> scriptClass, Binding context) throws InstantiationException, IllegalAccessException, InvocationTargetException, NoSuchMethodException {
    Script script;
    try {
        Constructor<? extends Script> constructor = scriptClass.getConstructor(Binding.class);
        script = constructor.newInstance(context);
    } catch (NoSuchMethodException e) {
        // Fallback for non-standard "Script" classes.
        script = scriptClass.getDeclaredConstructor().newInstance();
        script.setBinding(context);
    }
    return script;
}
```

通过反射创建对象时会加载执行 `GroovyObjectSupport.getDefaultMetaClass`;最终的问题在于 `org.codehaus.groovy.reflection.ClassInfo`定义了一个`static`类型的`GlobalClassSet`,在`GlobalClassSet`中会创建一个`ManagedConcurrentLinkedQueue`,队列中会一直追加通过`ClassInfo`包裹的`Script`实例信息。



分析到此可以基本得出由于`Script`的每次反射创建，队列中的实例数目一直在增加，垃圾回收时无法释放；而由于执行的脚本定义本身时固定的，变化的时运行时绑定的变量信息，可以考虑将 `Scripit` 实例信息进行缓存



##### 4、代码优化

```java
private final ConcurrentHashMap<String, Script> scriptCacheMap = new ConcurrentHashMap<>();  // 存储Scripit对象

public Object executeScript(String script) {
    var groovyShell = new GroovyShell();

    // 计算脚本 md5
    var scriptMd5 = genSourceCacheKey(scriptContent);

    // 此处不考虑并发的问题，并发访问时，多解析脚本内容，不会导致数据的不安全
    Script groovyScript;
    if (scriptCacheMap.containsKey(scriptMd5)) {
        groovyScript = scriptCacheMap.get(scriptMd5);
    } else {
        groovyScript = groovyShell.parse(scriptContent);
        scriptCacheMap.put(scriptMd5, groovyScript);
    }

    Object obj = groovyScript.run();
}

private String genSourceCacheKey(String script) {
    // 此处与groovy计算脚本md5保持一致
    try {
        return EncodingGroovyMethods.md5(script);
    } catch (NoSuchAlgorithmException e) {
        // TODO Auto-generated catch block
        throw new GroovyRuntimeException(e);
    }
}
```



此处调用`parse`解析字符串脚本时未加锁，即使在多线程并发场景，产生的结果仅仅为多解析几次，`Script`实例信息存在覆盖的情况，不存在数据不安全的问题。如果加锁还会存在性能损失问题，得不偿失。



#### 三、其他

##### 1、部分文章介绍由于脚本文件名称随即产生引起泄漏

答：实际定位测试时不存在该问题，每次创建`Script`对象前生成的文件名称都为 `Script1.groovy`，分析`GroovyShell`代码可以查看到获取文件名称方法如下：

```java
private final AtomicInteger counter = new AtomicInteger(0);

protected String generateScriptName() {
    return "Script" + counter.incrementAndGet() + ".groovy";
}
```



##### 2、生产环境和本地验证配置

|               | 生产环境 | 验证环境 |
| ------------- | -------- | -------- |
| `Spring Boot` | `2.0.6`  | `3.3.1`  |
| `JDK`         | `1.8`    | `21`     |
| `Groovy`      | `3.0.6`  | `4.0.22` |

`pom.xml`

```xml
<dependency>
    <groupId>org.apache.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>4.0.22</version>
    <type>pom</type>
</dependency>
```

