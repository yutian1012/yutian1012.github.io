---
title: activiti流程存储
tags: [activiti]
---

RepositoryService，流程存储服务组件。主要用于对Activiti中的流程存储的相关数据进行操作。包括流程存储数据的管理、流程部署以及流程的基本操作等。

1）DeploymentBuilder对象

```
DeploymentBuilder builder=repositoryService.createDeployment();
```

2）processDefinition对象

从流程部署中获取流程定义对象ProcessDefinition

```
ProcessDefinition def=repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
```