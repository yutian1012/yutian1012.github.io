---
title: activiti流程多实例
tags: [bpmx]
---

参考：http://blog.csdn.net/chq1988/article/details/41513451

1）多实例

多实例是在流程中重复执行一个特定步骤。跟程序开发的概念相比，多实例和foreach机制类似。流程引擎根据一个给定的集合，循环执行某一个步骤，甚至一个完整的子流程。

多实例节点是一个普通的活动节点，加上一个multiInstanceLoopCharacteristics子元素。这样就会使一个节点在运行时执行多次，执行方式可以是串行或并行。

2）将节点变成多实例节点

活动可以通过添加multiInstanceLoopCharacteristics变成多实例节点。

可变成多实例节点的流程节点：用户任务，脚本任务，Java节点，Web服务节点，业务规则任务，邮件任务，人工任务，内嵌子流程，外部子流程。

不可变成多实例节点的流程节点：事件节点（开始，结束）和网关节点不能作为多实例节点。

3）子实例的产生

主实例产生子实例的excution（执行流），都要提供如下变量：

nrOfInstances：实例总数

nrOfActiveInstances：当前活动的，比如，还没完成的，实例数量。 对于顺序执行的多实例，值一直为1。

nrOfCompletedInstances：已经完成实例的数目。

可以通过execution.getVariable(x)方法获得这些变量。

4）xml元素

要把一个节点设置为多实例，节点xml元素必须设置一个multiInstanceLoopCharacteristics子元素。isSequential属性表示节点是进行 顺序执行还是并行执行。

```
<multiInstanceLoopCharacteristics isSequential="false|true">
 ...
</multiInstanceLoopCharacteristics>
```

5）设置实例数目

方式一：使用loopCardinality元素直接指定

```
<multiInstanceLoopCharacteristics isSequential="false|true">
  <loopCardinality>5</loopCardinality>
</multiInstanceLoopCharacteristics>
```

方式二：通过loopDataInputRef子元素指定

```
<userTask id="miTasks" name="My Task ${loopCounter}" activiti:assignee="${assignee}">
  <multiInstanceLoopCharacteristics isSequential="false">
    <loopDataInputRef>assigneeList</loopDataInputRef>
    <inputDataItem name="assignee" />
  </multiInstanceLoopCharacteristics>
</userTask>
```

注：假设assigneeList变量包含这些值kermit，gonzo，foziee三个值。那么三个用户任务会同时创建。每个分支都会拥有一个用名为assignee的流程变量，这个变量会包含集合中的对应元素。

注2：loopDataInputRef和inputDataItem的缺点是名字不好记；根据BPMN2.0格式定义，它们不能包含表达式。

方式三：multiInstanceCharacteristics中设置collection和elementVariable属性

```
<userTask id="miTasks" name="My Task" activiti:assignee="${assignee}">
  <multiInstanceLoopCharacteristics isSequential="true"
     activiti:collection="${myService.resolveUsersForTask()}" activiti:elementVariable="assignee" >
  </multiInstanceLoopCharacteristics>
</userTask>
```

6）多实例节点的结束条件

多实例节点在所有实例都完成时才会结束。也可以指定一个表达式在每个实例结束时执行。如果表达式返回true，所有其他的实例都会销毁，多实例节点也会结束，流程会继续执行。这个表达式必须定义在completionCondition子元素中。

```
<userTask id="miTasks" name="My Task" activiti:assignee="${assignee}">
  <multiInstanceLoopCharacteristics isSequential="false"
     activiti:collection="assigneeList" activiti:elementVariable="assignee" >
    <completionCondition>${nrOfCompletedInstances/nrOfInstances >= 0.6 }</completionCondition>
  </multiInstanceLoopCharacteristics>
</userTask>
```