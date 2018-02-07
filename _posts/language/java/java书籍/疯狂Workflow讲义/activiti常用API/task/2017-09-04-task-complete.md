---
title: 任务声明与完成
tags: [activiti]
---

1）任务的声明

任务声明是指将任务分配到某个用户下，即将该用户作为任务的受理人，可以使用TaskService的claim方法进行任务受理人指定（效果类似于setAssignee方法）。即设置ACT_RU_TASK表的ASSIGNEE_字段的值为用户应的ID。

```
// 调用claim方法
taskService.claim(task.getId(), "1");
// 此处将会抛出异常
taskService.claim(task.getId(), "2");
```

注：与setAssignee方法不同的地方是，一旦调用了claim方法声明了任务的受理人，那么如果再次调用该方法将同一个任务分配到另外的用户下，则会抛出异常。如果两次传入的用户ID值相同，则不会抛出异常。

2）任务完成

主要提供了两种complete方法，一种只需要taskId，另外一种可以传递流程参数（Map参数集合），传入的参数可以作为执行流参数，其会作用与任务所在的执行流。

```
// 启动流程
ProcessInstance pi = runtimeService.startProcessInstanceById(pd.getId());
// 查找任务
Task task = taskService.createTaskQuery().processInstanceId(pi.getId())
    .singleResult();
// 调用complete方法完成任务，传入参数
Map<String, Object> vars = new HashMap<String, Object>();
vars.put("days", 2);
taskService.complete(task.getId(), vars);

// 再次查找任务
task = taskService.createTaskQuery().processInstanceId(pi.getId())
    .singleResult();

//得到参数
Integer days = (Integer)taskService.getVariable(task.getId(), "days");
if (days > 5) {
    System.out.println("大于5天，不批");
} else {
    System.out.println("小于5天，完成任务，流程结束");
    taskService.complete(task.getId());
}
```

注：在调用complete方法时，会将完成的Task数据从数据表中删除，如果发现这个任务为流程中的最后一个任务，则会连同流程实例的数据也一并删除。并且按照历史配置来记录流程的历史数据。