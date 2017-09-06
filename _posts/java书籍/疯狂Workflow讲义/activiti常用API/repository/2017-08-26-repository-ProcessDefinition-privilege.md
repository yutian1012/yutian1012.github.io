---
title: activiti流程定义权限
tags: [activiti]
---

主要实现流程中的用户或用户组与流程元素（包括流程定义、流程任务等）的权限关系。

在ACT_RU_IDENTITYLINK表中存在GROUP_ID_和USER_ID_分别记录相应的用户组和用户主键，存在TASK_ID_和PROC_DEF_ID_分别记录相应的立场任务和流程定义的主键。

### 用户和用户组权限设置

1）设置流程定义的用户权限

addCandidateStarterUser将流程定义与用户绑定，该方法实际上是向一个中间表ACT_RU_IDENTITYLINK中加入数据。

```
repositoryService.addCandidateStarterUser(def.getId(), user.getId());
```

注：第一个参数为流程定义ID，第二个参数为用户的ID。

![](/images/book/workflow/activiti/processdef/privilege/useridentitylink.png)

![](/images/book/workflow/activiti/processdef/privilege/user.png)

![](/images/book/workflow/activiti/processdef/privilege/procdef.png)

2）设置流程定义的用户组权限

addCandidateStarterGroup方法设置流程定义与用户组的权限

```
repositoryService.addCandidateStarterGroup(def.getId(), group.getId());
```

### IdentityLink对象

1）对象结构

IdentityLinkEntity对象的属性信息，对应表ACT_RU_IDENTITYLINK

```
protected String id;//对应ID_字段
  
protected String type;//数据类型，对应TYPE_字段，分别为assignee，candidate和owner。其中candidate为创建流程实例的请求者

protected String userId;//关联用户表主键

protected String groupId;//关联用户组表的主键

protected String taskId;/关联流程任务表主键

protected String processDefId;//关联流程定义表主键
```

2）查询权限数据

使用IdentityService获取流程权限数据。根据流程定义获取有权限申请的用户和用户组数据。

```
// 得到身份服务组件
IdentityService identityService = engine.getIdentityService();
//根据流程定义获取用户组数据
List<Group> groups = identityService.createGroupQuery().potentialStarter(def.getId()).list();
//根据流程定义获取用户数据
List<User> users = identityService.createUserQuery().potentialStarter(def.getId()).list();
```

使用repositoryService获取流程权限数据，根据用户获取该用户有权限启动的流程定义。

```
//得到流程存储服务实例
RepositoryService repositoryService = engine.getRepositoryService();
//设置权限数据
repositoryService.addCandidateStarterGroup(def.getId(), group.getId());
// 根据用户查询用权限的流程定义
List<ProcessDefinition> defs = repositoryService.createProcessDefinitionQuery()
    .startableByUser("user1").list();
// 根据流程定义查询全部的 IdentityLink（ACT_RU_IDENTITYLINK表） 数据
List<IdentityLink> links=repositoryService
    .getIdentityLinksForProcessDefinition(def.getId());
```