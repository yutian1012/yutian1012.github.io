---
title: ProcessEngine流程引擎
tags: [activiti]
---

Activiti提供了多种创建流程引擎的方式，可以通过ProcessEngineConfiguration的buildProcessEngine方法，也可以使用ProcessEngines的init方法来创建ProcessEngine实例。

### buildProcessEngine方法

ProcessEngineConfiguration类提供了buildProcessEngine方法来创建流程引擎。

```
// 创建Activiti配置对象
ProcessEngineConfiguration config = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("interceptor-config.xml");
// 初始化流程引擎
ProcessEngine engine = config.buildProcessEngine();
```

### ProcessEngines工具类

```
ProcessEngines.init();
// 得到ProcessEngines的Map
Map<String, ProcessEngine> engines = ProcessEngines.getProcessEngines();
```

init方法会读取Activiti默认配置文件（activiti.cfg.xml），然后创建ProcessEngine实例，并放置到engines的map对象中。默认map的key是default。

```
ProcessEngine processEngine=ProcessEngines.getDefaultProcessEngine();
Map<String,ProcessEngine> engines=ProcessEngines.getProcessEngines();
System.out.println(engines.size());
assert processEngine==engines.get("default");
```

注：ProcessEngines类的getDefaultProcessEngine会判断是否已经初始化了，如果没有初始化，就会调用init方法，然后再从map中获取default为key的ProcessEngine对象。

### ProcessEngine对象的注册和注销

ProcessEngine是有Map对象来管理的，因此可以向map对象中注册（registerProcessEngine方法），同时从map对象中注销（unregister方法）。这两个方法都是ProcessEngines类提供的。

unregister方法只是单纯地将ProcessEngine实例从map集合中移除，并不会调用ProcessEngine的close方法。

### ProcessEngine对象

1）常用服务类获取

ProcessEngine中会初始化一系列服务组件，这些服务组件提供了大部分控制流程引擎数据的业务方法，这些组件类似于J2EE的Service层组件，可以通过ProcessEngine类的getXXXService方法获取。

RepositoryService类，提供了一系列管理流程定义和流程部署的API

RuntimeService类，在流程运行时对流程实例进行管理与控制

TaskService类，对流程任务进行管理，如任务提醒、任务完成和任务分配等。

IdentityService类，提供对流程角色数据进行管理的API，这些角色数据包括用户组、用户以及它们之间的关系。

ManagementService类，提供对流程引擎进行管理和维护的服务

HistoryService类，对流程的历史数据进行操作，包括查询、删除等。

2）close方法

close方法会对流程引擎进行关闭操作，包括关闭工作执行器（JobExecutor），如果配置了databaseSchemaUpdate属性值为create-drop，会执行数据表删除操作。