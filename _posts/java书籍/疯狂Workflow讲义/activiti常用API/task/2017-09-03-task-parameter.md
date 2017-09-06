---
title: 任务参数
tags: [activiti]
---

在Activiti中参数类型分为流程参数和任务参数。activiti中会将各种参数保存到act_ru_variable数据表中。使用taskService的setVariable来设参数。

1）流程参数类型

流程中会涉及到流程参数类型，在act_ru_variable表中提供了DOUBLE_，LONG_，TEXT_和TEXT2_字段来存储变量的值。

```
Date d = new Date();
short s = 3;
//设置各种基本类型参数
taskService.setVariable(task1.getId(), "arg0", false);
taskService.setVariable(task1.getId(), "arg1", d);
taskService.setVariable(task1.getId(), "arg2", 1.5D);
taskService.setVariable(task1.getId(), "arg3", 2);
taskService.setVariable(task1.getId(), "arg4", 10L);
taskService.setVariable(task1.getId(), "arg5", null);
taskService.setVariable(task1.getId(), "arg6", s);
taskService.setVariable(task1.getId(), "arg7", "test");
```

![](/images/book/workflow/activiti/task/variable/primary_variable.png)

存储规则：

除了Double，Nulll和字符串类型外，都会将数据保存到数据表的LONG_字段中。int，long，short也会将值存入到数据表的TEXT_字段中。因此，这三个类型会同时存在与LONG_和TEXT_字段中。

double会将执行存储在DOUBLE_字段中

String会将值存储在TEXT_字段中

注：如果设置的参数名称存在重复，那么后一参数将会覆盖前一个设置的参数。

2）序列化参数

在设置参数是如果传递的不是基本数据类型，而是对象类型的数据时，会将该对象序列化（被序列化的对象必须实现serializable接口），然后保存到ACT_GE_BYTEARRAY表中，并设置act_ru_variable表的bytearray_id_字段关联。

```
taskService.setVariable(task1.getId(), "arg0", new TestVO("crazyit"));
```

注：TestVO是自定义对象，且该类实现了Serializable接口。

![](/images/book/workflow/activiti/task/variable/serializable_variable1.png)

![](/images/book/workflow/activiti/task/variable/pserializable_variable2.png)

3）获取参数

使用TaskService类的getVariable方法获取任务参数，需要传递taskId和参数名。

```
Integer days = (Integer)taskService.getVariable(task1.getId(), "days");
```

注：在获取参数时必须知道参数类型，这里需要强制类型转换。

4）参数作用域

当任务与流程绑定后（前提条件），设置的参数均会有作用域。使用setVariable方法设置的参数在整个流程中都能够有效（act_ru_variable表的task_id_字段为空，proc_inst_id_字段为流程实例的主键）。使用setVariableLocal方法设置的参数仅仅在当前这个任务中有效（act_ru_variable表的task_id_字段为当前任务id）。

```
taskService.setVariable(task.getId(), "days", 10);
taskService.setVariableLocal(task.getId(), "target", "欧洲");
```

getVariable方法能够获取setVariable方法和setVariableLocal方法设置的参数。而getVariableLocal方法只能查询到setVariableLocal设置的参数。

```
// 获取参数
Object data1 = taskService.getVariable(task.getId(), "days");
System.out.println("获取休假天数：" + data1);
//获取setVariableLocal设置的参数
Object data2 = taskService.getVariable(task.getId(), "target");
System.out.println("获取目的地： " + data2);
//getVariableLocal方法不能获取setVariable方法设置的参数
Object data3 = taskService.getVariableLocal(task.getId(), "days");
System.out.println("使用getVariableLocal方法获取天数：" + data3);
```

![](/images/book/workflow/activiti/task/variable/setvariable.png)

注：如果任务不与流程绑定，那么不管调用setVariable还是调用setVariableLocal方法，均会设置task_id_字段值，而与流程绑定后，setVariable方法不会再设置task_id_字段值。

5）设置多个参数

设置多个参数可以定义一个序列化对象，将这些参数放到这个对象中，通过序列化参数的方式设置。也可以调用TaskService类的setVariables和setVariablesLocal方法传递Map集合（多个参数放置在map集合中）。

```
//初始化参数
Map<String,Object> vars = new HashMap<String, Object>();
vars.put("days", 10);
vars.put("target", "欧洲");
taskService.setVariables(task1.getId(), vars);
```

注：setVariables比setVariable方法末尾多了一个s

![](/images/book/workflow/activiti/task/variable/setvariables.png)

注：map集合中的参数会拆分出多条记录存放。即setVariables方法最终会变量参数的map集合，然后在逐一设置参数。