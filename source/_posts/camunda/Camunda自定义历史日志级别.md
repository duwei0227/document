---
layout: post
title: Camunda自定义历史日志级别
categories: [Camunda]
permalink: camunda/custom-history-level.html
date: 2024-05-10 14:56:29
---

`Camunda 7`支持的日志级别为：`None`、`Activity`、`Audit`、`Full`，记录的信息逐级增加，`None`和`Activity`不会记录流传变量，**可以显著提升引擎性能**，如果在项目中有诉求需要对历史日志自定义记录，也可以通过实现`HistoryLevel`接口或者继承已有的历史日志类进行能力扩展。



自定义日志需要通过继承 `AbstractProcessEnginePlugin` 类进行注册，相关示例如下：

```java
@Component
public class CustomHistoryLevelProcessEnginePlugin extends AbstractProcessEnginePlugin {

  public void preInit(ProcessEngineConfigurationImpl processEngineConfiguration) {
    List<HistoryLevel> customHistoryLevels = processEngineConfiguration.getCustomHistoryLevels();
    if (customHistoryLevels == null) {
      customHistoryLevels = new ArrayList<HistoryLevel>();
      processEngineConfiguration.setCustomHistoryLevels(customHistoryLevels);
    }
    customHistoryLevels.add(CustomVariableHistoryLevel.getInstance());

  }
}
```



**在Spring Boot**项目中，需要将`CustomHistoryLevelProcessEnginePlugin` 注册为`Bean`



通过接口实现日志事件类型判断：

```java
public class CustomVariableHistoryLevel implements HistoryLevel {

  public static final CustomVariableHistoryLevel INSTANCE = new CustomVariableHistoryLevel();

  public static CustomVariableHistoryLevel getInstance() {
    return INSTANCE;
  }

  public int getId() {
      // 注意：不要和camunda已有的冲突
    return 11;
  }

  public String getName() {
    return "custom-variable";
  }

  public boolean isHistoryEventProduced(HistoryEventType historyEventType, Object entity) {
     // 可以对事件类型进行判断，支持的事件类型可以查看 HistoryEventTypes
  }

}
```



也可以通过继承已有的日志类进行能力扩展：

```java
public class CustomVariableHistoryLevel extends HistoryLevelActivity {
    // 同实现接口一样
}
```



参考：[https://github.com/camunda/camunda-bpm-examples/blob/master/process-engine-plugin/custom-history-level/README.md](https://github.com/camunda/camunda-bpm-examples/blob/master/process-engine-plugin/custom-history-level/README.md)