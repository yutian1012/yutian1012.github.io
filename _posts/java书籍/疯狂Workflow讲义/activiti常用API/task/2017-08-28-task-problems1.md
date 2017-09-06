---
title: 流程任务与历史任务的关系
tags: [activiti]
---

历史任务数据是何时生成的？

创建任务，在执行完成下面的代码后会在数据库中的act_ru_task和act_hi_taskinst表中存入数据。

```
public static void main(String[] args) {
    // 创建流程引擎
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    //获取任务服务组件
    TaskService taskService = engine.getTaskService();
    //保存第一个Task，不设置ID
    Task task1 = taskService.newTask();
    taskService.saveTask(task1);
    //保存第二个Task，设置ID
    Task task2 = taskService.newTask("审核任务");
    taskService.saveTask(task2);
}
```

执行完成saveTask后，发现数据库中act_ru_task表中多了两条数据，而act_hi_taskinst表中却没有数据。main方法结束后，才能在act_hi_taskinst表中看到数据。