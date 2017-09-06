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


