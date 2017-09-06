---
title: activiti流程变量
tags: [bpmx]
---

1）API

activiti中TaskService，RuntimeService，TaskEntity以及executionEntity类都提供了设置参数的方法。

```
org.activiti.engine.TaskService
org.activiti.engine.RuntimeService
org.activiti.engine.impl.persistence.entity.TaskEntity
org.activiti.engine.impl.persistence.entity.ExecutionEntity
```

这四个类中都提供了setVarialbe，setVariableLocal，setVariables，setVariablesLocal方法，用来设置流程参数。

2）流程变量作用域

方式一：getVariableLocal方法。

task（TaskEntity）调用getVariableLocal方法，获取的变量范围是属于任务ID的变量，通过taskId到act_ru_variable表中匹配记录，任务消失，这个变量也就删除掉。

execution（ExecutionEntity）调用getVariableLocal方法，获取的变量和当前的execution相关。在数据库表act_ru_variable中查询流程运行实例的executionId查询流程变量。

方式二：getVariable方法（取全局的变量）

task（TaskEntity）调用getVariable方法，获取流程全局变量，首先他会根据任务id查询流程变量。如果找不到则根据任务的executionId进行查找，如果再查询不到，则查询executionId的父流程实例，如果查询不到则继续往上查找。

execution（ExecutionEntity）调用getVariable方法，获取流程全局变量。首先会去变量表中查询对应流程运行实例的流程变量，如果找不到在查询父级流程实例id和变量名查询，如果查询不到则继续网上查找。

注：一旦任务与流程实例进行了关联，task.setVariable方法就不会将taskId存放到act_ru_variable表的task_id_字段中了。

3）代码

```
// 获取流程引擎实例
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
// 获取任务服务组件
TaskService taskService = engine.getTaskService();
// 获取运行服务组件
RuntimeService runtimeService = engine.getRuntimeService();
// 流程存储服务组件
RepositoryService repositoryService = engine.getRepositoryService();
// 部署流程描述文件
Deployment dep = repositoryService.createDeployment()
        .addClasspathResource("bpmn/vacation.bpmn").deploy();
// 查找流程定义
ProcessDefinition pd = repositoryService.createProcessDefinitionQuery()
        .deploymentId(dep.getId()).singleResult();
// 启动流程
ProcessInstance pi = runtimeService.startProcessInstanceById(pd.getId());

// 分别调用setVariable和setVariableLocal方法
Task task = taskService.createTaskQuery().processInstanceId(pi.getId())
     .singleResult();

ExecutionEntity execution=(ExecutionEntity) runtimeService.createExecutionQuery().executionId(task.getExecutionId()).singleResult();

runtimeService.setVariable(execution.getId(), "testruntimeService", 1);
runtimeService.setVariableLocal(execution.getId(), "testLocalruntimeService", 1);

/*
 * runtimeService内部调用了executionEntity的变量设置方法--查看SetExecutionVariablesCmd
 * execution.setVariable("testExecutionEntity", 2);
execution.setVariableLocal("testLocalExecutionEntity", 2);*/

taskService.setVariable(task.getId(), "testTaskService", 4);
taskService.setVariableLocal(task.getId(), "testLocalTaskService", 4);

/*
 * taskService内部调用了TaskEntity的变量设置方法--查看SetTaskVariablesCmd
 * ((TaskEntity)task).setVariable("testTask", 3);
((TaskEntity)task).setVariable("testLocalTask", 3);*/

taskService.complete(task.getId());//设置断点
        
//查询第二个任务
task = taskService.createTaskQuery().processInstanceId(pi.getId()).singleResult();

execution=(ExecutionEntity) runtimeService.createExecutionQuery().executionId(task.getExecutionId()).singleResult();

runtimeService.setVariable(execution.getId(), "testruntimeService2", 5);
runtimeService.setVariableLocal(execution.getId(), "testLocalruntimeService2", 5);

taskService.setVariable(task.getId(), "testTaskService2", 6);
taskService.setVariableLocal(task.getId(), "testLocalTaskService2", 6);
```

注：在taskService.complete方法处设置断点

![](/images/work/bpmx/source/setvariable.png)

执行完成taskService.complete后，setVariableLocal方法设置的变量就被删除了。执行完成第二个任务后又多了4条数据，且execution_id_都相同（不同任务的执行流ID是相同的）。

![](/images/work/bpmx/source/setvariable2.png)

注：当流程按照规则只执行一次的时候，那么流程实例就是执行对象。一个流程中，执行对象可以存在多个，但是流程实例只能有一个。