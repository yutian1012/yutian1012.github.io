---
title: 流程实例启动
tags: [activiti]
---

### 启动流程

1）方式一：

通过processDefinitionId（流程定义主键）启动，启动时可以传递业务主键，传递流程参数。

```
// 创建流程引擎
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
// 得到流程存储服务实例
RepositoryService repositoryService = engine.getRepositoryService();
RuntimeService runtimeService = engine.getRuntimeService();
// 部署流程描述文件
Deployment dep = repositoryService.createDeployment()
    .addClasspathResource("bpmn/startById.bpmn20.xml").deploy();        
// 查找流程定义
ProcessDefinition pd = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
//设置流程参数
Map<String, Object> vars = new HashMap<String, Object>();
vars.put("days", 5);
//启动流程
runtimeService.startProcessInstanceById(pd.getId());
//传递流程参数
runtimeService.startProcessInstanceById(pd.getId(), vars);
//传递业务主键businessKey
runtimeService.startProcessInstanceById(pd.getId(), "vacationRequest1");
//传递业务主键和流程参数
runtimeService.startProcessInstanceById(pd.getId(), "vacationRequest2", vars);
// 查询流程实例，结果为4
long count = runtimeService.createProcessInstanceQuery().count();
```

![](/images/book/workflow/activiti/execution/startprocess1.png)

注：同一个流程下，业务主键必须唯一，因此第二个业务主键传递的值与第一个不同。

2）方式二：

通过流程定义中的key来启动，可以根据流程描述文件中定义的process节点的id属性来启动流程。该方式同样提供了4中启动方法。

```
runtimeService.startProcessInstanceByKey("vacationRequest");
runtimeService.startProcessInstanceByKey("vacationRequest", vars);
runtimeService.startProcessInstanceByKey("vacationRequest", "testKey");
runtimeService.startProcessInstanceByKey("vacationRequest", "testKey2", vars);
```

注：可以查看相应的流程定义文件xml，其process节点下的id值为vacationRequest。

3）方式三：

根据消息事件启动流程。在实际应用的过程中，可能出现以下场景：一个订单流程，当接收到相应订单信息后，会自动启动该流程。

解决方案：使用activiti的消息事件，在流程定义的开始事件中添加消息事件定义，当流程接收到某个消息事，就启动流程。

流程定义文件：

```
<!-- 定义消息 -->
<message id="startMsg" name="startMsg"></message>

<process id="saleOrder" name="saleOrder">
    <startEvent id="start" >
        <!-- 定义消息事件 -->
        <messageEventDefinition messageRef="startMsg"></messageEventDefinition >
    </startEvent>
    <sequenceFlow sourceRef="start" targetRef="confirmOrder" />
    <userTask id="confirmOrder"></userTask>     
    <sequenceFlow sourceRef="confirmOrder" targetRef="sendGoods" />     
    <userTask id="sendGoods"></userTask>
    <sequenceFlow sourceRef="sendGoods" targetRef="end" />
    <endEvent id="end" />
</process>
```

注：message节点为消息事件的定义，另外，在开始节点（startEvent）处引用消息事件。

执行原理：

activiti在家中流程描述文件时，如果发现开始事件中存在消息事件，则会向act_ru_event_subscr表写入事件描述的数据。在消息事件发布后，会有对应的监听，在监听中会查询act_ru_event_subscr表，并获取configuration_字段值（该字段值保存的是流程定义的ID），启动相关的流程。

启动方法，该方式也提供了4种启动方法。

```
// 部署流程描述文件
repositoryService.createDeployment()
    .addClasspathResource("bpmn/startByMessage.bpmn20.xml").deploy();   
//初始化流程参数
Map<String, Object> vars = new HashMap<String, Object>();
vars.put("days", 4);
//启动流程，传递消息名称
runtimeService.startProcessInstanceByMessage("startMsg");
runtimeService.startProcessInstanceByMessage("startMsg", vars);
runtimeService.startProcessInstanceByMessage("startMsg", "testKey");
runtimeService.startProcessInstanceByMessage("startMsg", "testKey2", vars);
```

注：这里并没有通过事件机制来发布，而是简单的调用方法启动流程。查看事件相关章节了解具体的执行过程。

注2：消息名称对应的是message节点的name属性，而不是id属性。而消息事件定义（messageEventDefinition）的messageRef属性引用的是message节点的id属性。

### ExecutionEntity的创建

ProcessDefinitionEntity类的createProcessInstance方法，进而会调用newProcessInstance方法。

```
@Override
protected InterpretableExecution newProcessInstance(ActivityImpl activityImpl) {
    ExecutionEntity processInstance = new ExecutionEntity(activityImpl);
    processInstance.insert();//保存内容到数据库中
    return processInstance;
}
```

ProcessDefinitionEntity的继承体系，该类继承了ProcessDefinitionImpl类，大部分的操作都在ProcessDefinitionImpl中实现了，ProcessDefinitionEntity覆盖了newProcessInstance的实现，因此，实际执行时执行的是ProcessDefinitionEntity中的newProcessInstance方法。

![](/images/book/workflow/activiti/execution/ProcessDefinitionEntity.png)