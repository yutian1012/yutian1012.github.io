---
title: activiti实体类服务组件和实体类
tags: [activiti]
---

### 实体类服务组件

activiti对数据的管理都会提供相应的服务组件。

1）IdentityService，对用户和用户组的管理

### 实体类

activiti中每个实体的实现类均有字节的接口，并且每个实体的实现类名称均为XXXEntity。如Group接口与GroupEntity实现类。

Activiti没有提供任何实体实现类（XXXEntity）的API，如果需要创建这些实体对象，只需调用相应业务组件的方法即可。如创建Group对象，调用indentityService的newGroup方法即可。