---
title: activiti实体类服务组件和实体类
tags: [activiti]
---

### 实体类服务组件

activiti对数据的管理都会提供相应的服务组件。

1）IdentityService，对用户和用户组的管理

2）RepositoryService，流程存储服务组件。主要用于对Activiti中的流程存储的相关数据进行操作。包括流程存储数据的管理、流程部署以及流程的基本操作等。

3）TaskService，提供了操作流程任务的API，包括任务的查询、创建与删除、权限设置和参数设置等。

4）RuntimeService主要用于管理流程在运行时产生的数据以及提供对流程操作的API。

其中流程运行时产生的数据包括流程参数、事件、流程实例以及执行流等。流程的操作包括开始流程、让流程前进等。

### 实体类

activiti中每个实体的实现类均有字节的接口，并且每个实体的实现类名称均为XXXEntity。如Group接口与GroupEntity实现类。

Activiti没有提供任何实体实现类（XXXEntity）的API，如果需要创建这些实体对象，只需调用相应业务组件的方法即可。如创建Group对象，调用indentityService的newGroup方法即可。

### 实体类的查询对象

Activiti中的每个实体对象都对应了查询对象（XXXQuery），具有自己相应的查询方法和排序方法。