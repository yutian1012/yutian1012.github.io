---
title: 任务权限
tags: [activiti]
---

任务可以设置角色的权限数据，这些角色包括用户组和用户。

任务的权限数据设置与流程权限设置类似，也是使用ACT_RU_IDENTITYLINK表，该表中存在GROUP_ID_和USER_ID_分别记录相应的用户组和用户主键，存在TASK_ID_记录相应的立场任务主键。

1）设置候选用户组

```
TaskService taskService = engine.getTaskService();
//保存第一个Task
Task task1 = taskService.newTask("task1");
taskService.saveTask(task1);
//保存第二个Task
Task task2 = taskService.newTask("task2");
taskService.saveTask(task2);
//绑定用户组和任务关系
taskService.addCandidateGroup("task1", groupA.getId());
taskService.addCandidateGroup("task2", groupA.getId());
```

注：任务数据task1会存放到act_ru_task表的ID_字段中，权限数据会存放到act_ru_identitylink表中。

![](/images/book/workflow/activiti/task/identity/group_identity.png)

2）设置候选用户

任务拥有者与候选者的区别：候选用户是一群将会拥有或执行任务权限的人，但任务只允许一个人执行或者拥有。

```
// 获取任务服务组件
TaskService taskService = engine.getTaskService();
//保存一个Task
Task task1 = taskService.newTask("task1");
taskService.saveTask(task1);
//绑定用户与任务关系
taskService.addCandidateUser("task1", user.getId());
taskService.addCandidateUser("task1", user2.getId());
```

![](/images/book/workflow/activiti/task/identity/user_identity.png)

3）查询数据权限

taskService类的createTaskQuery获取TaskQuery对象，该对象提供了taskCandidateGroup方法和taskCandidateUser方法可实现根据用户和用户组的ID查询task数据。

```
//根据候选组查询task
List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup(groupA.getId()).list();
//根据用户查询task
tasks = taskService.createTaskQuery().taskCandidateUser(user.getId()).list();
```

查询权限关系数据，可以根据task的ID信息查询IdentityLink权限数据集合

```
List<IdentityLink> links = taskService.getIdentityLinksForTask(tasks.get(0).getId());
```

4）设置任务持有人

taskService类的setOwner方法来设置任务的持有人。会设置act_ru_task表的owner_字段为相应的用户ID。同时act_hi_taskinst表的owner_字段也会设置该值。

```
taskService.setOwner(task1.getId(), user.getId());
```

如果想根据用户查询其所拥有的任务，则可以调用taskQuery的taskOwner方法。

```
List<Task> tasks=taskService.createTaskQuery().taskOwner(user.getId()).list();
```

5）设置任务受理人

taskService类的setAssignee方法用于设置任务的受理人，该方法会将act_ru_task表的assignee_字段设置为用户的ID。

```
taskService.setAssignee(task1.getId(), user.getId());
```

如果想根据受理人查询任务，可以调用taskService类的taskAssignee方法查询。

```
List<Task> tasks=taskService.createTaskQuery().taskAssignee(user.getId()).list();
```

6）候选人，任务受理人和任务持有人的关系：

候选人---> 受理人

候选人/候选组（candidate），可以执行任务的一类人或者多个组，候选人/候选组中都可以去签收任务，一旦某人签收，就成为受理人，其他人就不能再签收受理此任务。

持有人（owner）：持有人设置主要是存入历史表中，用于历史任务的查询。

注：任务的持有用户和受理用户只能有一个，而任务的候选用户/用户组可以有多个。

7）操作IdentityLink对象设置权限

taskService中提供了addGroupIdentityLink和addUserIdentityLink方法，用来设置任务权限数据。

```
taskService.addGroupIdentityLink(task1.getId(), groupA.getId(), IdentityLinkType.CANDIDATE);
//调用addUserIdentityLink方法
taskService.addUserIdentityLink(task1.getId(), user.getId(), IdentityLinkType.CANDIDATE);
taskService.addUserIdentityLink(task1.getId(), user.getId(), IdentityLinkType.OWNER);
taskService.addUserIdentityLink(task1.getId(), user.getId(), IdentityLinkType.ASSIGNEE);
```

注：类型是通过IdentityLinkType提供的常量来实现的。另外，开发者也可以根据用户自定义的数据标识来设置不同的权限。

查询IdentityLink数据，需要通过Task对象的Id来查询。

```
List<IdentityLink> links = taskService.getIdentityLinksForTask(task1.getId());
```

8）删除用户组权限

用户组的设置可通过taskService的addCandidateGroup方法或addGroupIdentityLink方法设置权限，同样，删除也提供了两种方法来实现。deleteGroupIdentityLinkf方法和deleteCandidateGroup方法删除用户组权限

```
taskService.deleteGroupIdentityLink(task1.getId(), groupA.getId(),IdentityLinkType.CANDIDATE);

taskService.deleteCandidateGroup(task1.getId(), groupA.getId());
```

9）删除用户权限

与删除用户组任务权限类似，TaskService中提供了两个方法来删除用户任务权限，deleteCandidateUser和deleteUserIdentityLink方法实现。

```
taskService.deleteCandidateUser(task1.getId(), user.getId());

taskService.deleteUserIdentityLink(task1.getId(), user.getId(), IdentityLinkType.OWNER);
```