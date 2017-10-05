---
title: activiti流程存储
tags: [activiti]
---

RepositoryService，流程存储服务组件。主要用于对Activiti中的流程存储的相关数据进行操作。包括流程存储数据的管理、流程部署以及流程的基本操作等。

1）DeploymentBuilder对象

```
DeploymentBuilder builder=repositoryService.createDeployment();

Deployment dep=builder.addClasspathResource("artifact/GetResource.txt").deploy();
```

注：调用deploy方法部署，返回Deployment对象。

2）processDefinition对象

从流程部署中获取流程定义对象ProcessDefinition

```
ProcessDefinition def=repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
```

中止和激活流程定义

```
// 调用suspendProcessDefinitionById中止流程定义 
repositoryService.suspendProcessDefinitionById(def.getId());
// 调用activateProcessDefinitionById激活流程定义 
repositoryService.activateProcessDefinitionById(def.getId());
```

3）设置用户和用户组的流程定义权限

只有用户或用户组拥有流程定义权限才能启动流程

```
//设置用户的流程定义权限
repositoryService.addCandidateStarterUser(def.getId(), user.getId());
//设置用户组的流程定义权限
repositoryService.addCandidateStarterGroup(def.getId(), group.getId());
```

注：会向ACT_RU_IDENTITYLINK表中存入权限信息。

查询获取权限

```
// 根据用户查询用权限的流程定义
List<ProcessDefinition> defs = repositoryService.createProcessDefinitionQuery()
    .startableByUser("user1").list();
// 根据流程定义查询全部的 IdentityLink（ACT_RU_IDENTITYLINK表） 数据
List<IdentityLink> links=repositoryService
    .getIdentityLinksForProcessDefinition(def.getId());
```

4）创建查询对象

查询部署资源：

```
InputStream is = repositoryService.getResourceAsStream(dep.getId(), 
    "artifact/GetResource.txt");
```

注：获取资源时需要传递部署资源的名称。

查询流程文件：

```
InputStream is = repositoryService.getProcessModel(def.getId());
```

注：获取流程描述文件的xml内容。