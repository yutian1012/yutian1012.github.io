---
title: 流程实例参数
tags: [activiti]
---

### 流程参数

activiti使用act_ru_variable表来保存流程中的参数，这些参数会有字节的生命周期和作用域。流程参数可以设置为作用域整个流程，也可以设置为只作用于当前任务（参考任务参数的设置）。

注：TaskService提供了设置流程任务参数方法，RuntimeService提供了设置流程参数的方法。

1）设置参数

RuntimeService的setVariable方法和stVariableLocal方法，允许传入基本类型参数，也允许传入序列化对象。其基本使用方法与TaskService的设置参数方法类似。设置参数时需要传递执行流的ID

```
ProcessInstance pi = runtimeService.startProcessInstanceByKey("vacationRequest");
//查询流程实例的执行流
Execution exe = runtimeService.createExecutionQuery().processInstanceId(pi.getId()).singleResult();
//设置流程参数
runtimeService.setVariable(exe.getId(), "days", 5);
```

注：为了区分流程实例与执行流，应该在到数据库中进行一次执行流的查询。

2）参数作用域

RuntimeService的setVariableLocal方法设置的参数只会作用于设置的执行流，一旦执行流结束，那么该参数将会失效。

RuntimeService的setVariable方法设置的参数作用域整个流程（包括子执行流）。

getVariableLocal方法无法获取setVariable方法设置的参数。

getVariable方法可以获取setVariableLocal方法设置的参数。

```
//启动流程
ProcessInstance pi = runtimeService.startProcessInstanceByKey("vacationRequest");
//查询全部的任务，得到相应的执行流，设置不同的参数
List<Task> tasks = taskService.createTaskQuery().processInstanceId(pi.getId()).list();

//设置流程参数（两个并行网关节点，分别设置Local参数和全局参数）
for (Task task : tasks) {
    Execution exe = runtimeService.createExecutionQuery()
            .executionId(task.getExecutionId()).singleResult();
    if ("Manager Audit".equals(task.getName())) {
        //经理审核节点，设置Local参数
        runtimeService.setVariableLocal(exe.getId(), "managerVar", "manager var");
    } else {
        //HR审核节点，设置全局参数
        runtimeService.setVariable(exe.getId(), "hrVar", "hr var");
    }
}
//查询并输出子执行流参数
for (Task task : tasks) {
    Execution exe = runtimeService.createExecutionQuery()
            .executionId(task.getExecutionId()).singleResult();
    //getVariable能够获取Local参数
    if ("Manager Audit".equals(task.getName())) {               
        System.out.println("使用getVariableLocal方法获取经理参数：" + 
                runtimeService.getVariableLocal(exe.getId(), "managerVar"));
        System.out.println("使用getVariable方法获取经理参数：" + 
                runtimeService.getVariable(exe.getId(), "managerVar"));
    } else {
        //getVariableLocal方法不能获取全局参数
        System.out.println("使用getVariableLocal方法获取HR参数：" + 
                runtimeService.getVariableLocal(exe.getId(), "hrVar"));
        System.out.println("使用getVariable方法获取HR参数：" + 
                runtimeService.getVariable(exe.getId(), "hrVar"));
    }
}
//完成任务，销毁Local参数
for (Task task : tasks) {
    taskService.complete(task.getId());
}
System.out.println("========  完成流程分支后     ========");
//重新查找流程任务，查看Local设置的参数是否被销毁
tasks = taskService.createTaskQuery().processInstanceId(pi.getId()).list();
for (Task task : tasks) {
    System.out.println("当前流程节点：" + task.getName());
    Execution exe = runtimeService.createExecutionQuery()
        .executionId(task.getExecutionId()).singleResult();
    //Local参数以销毁，输出null
    System.out.println("经理参数：" + runtimeService.getVariable(exe.getId(), "managerVar"));
    //全局参数未销毁（流程结束后才会销毁），可以正常输出
    System.out.println("HR参数：" + runtimeService.getVariable(exe.getId(), "hrVar"));
}
```

注：执行过程中，首先获取任务，然后根据任务获取其执行流ID，通过执行流ID设置流程变量。

注2：使用setVariableLocal方法设置的参数，只要执行流存在，就能够获取执行流对应的变量内容。

流程实例

![](/images/book/workflow/activiti/execution/executionParallel.png)

流程实例变量

![](/images/book/workflow/activiti/execution/executionVariables.png)

