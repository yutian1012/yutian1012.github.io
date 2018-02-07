---
title: 流程操作触发信号/消息事件
tags: [activiti]
---

### 事件

BPMN2.0规范中主要有两种类型的事件：捕获事件（Catching）和抛出事件（Throwing）。

1）Catching事件

在一个流程中定义了Catching事件后，流程执行到该节点时，会一直等待事件被捕获，流程才继续向下执行。

当流程到达信号Catching事件时，ACT_RU_EVENT_SUBSCR表会产生相应的事件描述数据。

2）抛出事件（Throwing）

在一个流程中定义了Throwing事件后，当流程执行到该节点时，该事件节点会自动往下执行，并不需要任何的信号指示。

### signalEventReceived抛出信号--触发信号事件

1）事件节点的定义

信号事件节点使用signal来定义，使用signalEventDefinition来定义信号事件，该元素可以定义在startEvent和intermediateCatchEvent元素中。

在一个流程中定义了Catching事件后，流程执行到该节点时，会一直等待触发的信号，直到接收到相应的信号，流程才继续向下执行。

当流程到达信号Catching事件时，ACT_RU_EVENT_SUBSCR表会产生相应的事件描述数据。对应的EVENT_TYPE_字段值为signal。

在Catching事件中，如果希望流程继续往下执行，可以使用RuntimeService的signalEventReceived方法抛出信号，那么Catching事件节点就会捕获到相应的信号，让流程继续往下执行。

2）方法

signalEventReceived方法有4个重载方法，使用时都需要传递相应的信号名称，此外，还可以传递执行流Id（则抛出的信号仅仅作用于某个执行流），传递流程参数（参数作用于整个流程）。

3）流程文件定义

```
<signal id="testSignal" name="testSignal"></signal>
<process id="testProcess" name="testProcess" isExecutable="true">
    <startEvent id="startevent1" name="Start"></startEvent>
    <!--定义捕获Catching事件-->
    <intermediateCatchEvent id="signalCatchEvent" name="SignalCatchEvent">
      <signalEventDefinition signalRef="testSignal"></signalEventDefinition>
    </intermediateCatchEvent>
...
</signal>
```

4）流程图：

![](/images/book/workflow/activiti/execution/catchingsignalgraph.png)

```
// 开始流程
ProcessInstance pi = runtimeService
    .startProcessInstanceByKey("testProcess");

List<Execution> exes = runtimeService.createExecutionQuery()
    .processInstanceId(pi.getId()).list();
System.out.println("流程在第一个任务，执行流数量为：" + exes.size());       
// 查找执行流
Execution exe = runtimeService.createExecutionQuery()
    .processInstanceId(pi.getId()).singleResult();

// 往下执行，将遇到Catch事件节点
runtimeService.signal(exe.getId()); 
exes = runtimeService.createExecutionQuery()
    .processInstanceId(pi.getId()).list();
System.out.println("流程到了Catch事件节点，等待触发，执行流数量：" + exes.size());  
//抛出一个信号，传递相关的signal名称（信号名称）
runtimeService.signalEventReceived("testSignal");
exes = runtimeService.createExecutionQuery()
    .processInstanceId(pi.getId()).list();
```

注：流程遇到Catching事件节点时会产生对应的执行流。

5）执行结果

在执行定义signal方法时设置断点，查看数据库的执行流信息

![](/images/book/workflow/activiti/execution/catchingsignal1.png)

执行signal方法，流程执行到catching事件处。此时会出现两个执行流第一个执行流的is_ACTIVITI_字段变为0（不活动状态），且REV_字段值变为2（对该记录进行了修改），需要等待Catching节点执行完毕。

![](/images/book/workflow/activiti/execution/catchingsignal2.png)

![](/images/book/workflow/activiti/execution/signalcatching.png)

signalEventReceived触发事件，Catching生成的执行流结束，第一个执行流执行到下一个节点

![](/images/book/workflow/activiti/execution/catchingsignal3.png)

### messageEventReceived方法抛出消息--触发消息事件

1）事件节点定义

消息事件节点使用message来定义，使用messageEventDefinition来定义消息事件，该元素可以定义在startEvent和intermediateCatchEvent元素中。

如果在startEvent中定义了消息事件，就可以使用RuntimeService的startProcessInstanceByMessage方法来启动的流程。

如果在intermediateCatchEvent元素下定义了消息事件，那么表示该消息事件节点是一个Catching事件，当流程执行到该节点时，会一直等待触发的消息，直到收到触发消息后，执行流才会继续往下执行。

流程到达消息Catching事件时，会在ACT_RU_EVENT_SUBSCR表中产生事件描述，对应的EVENT_TYPE_字段值为message。

messageEventDefinition元素不能放到intermediateThrowEvent元素下，即messageEventDefinition元素不允许定义到Throwing事件中，否则将抛出异常。

2）方法

messageEventReceived方法有两个重载，在使用时都需要传递消息名称和执行流ID，另外可以传递相关的流程变量（作用于整个流程实例）

3）流程文件定义

```
<message id="testMsg" name="testMsg"></message>

<process id="testProcess" name="testProcess">
    <startEvent id="startevent1" name="Start"></startEvent>
    <userTask id="usertask1" name="End Task"></userTask>
    <intermediateCatchEvent id="messageintermediatecatchevent1"
        name="MessageCatchEvent">
        <messageEventDefinition messageRef="testMsg"></messageEventDefinition>
    </intermediateCatchEvent>
...
</process>
```

4）流程图

![](/images/book/workflow/activiti/execution/catchingmessagegraph.png)

```
// 开始流流程
runtimeService.startProcessInstanceByKey("testProcess");
// 查询流执行流
List<Execution> exes = runtimeService.createExecutionQuery()
        .processDefinitionKey("testProcess").list();
System.out.println("遇到消息事件节点，执行流数量：" + exes.size());
// 根据事件名称查询当前事件所在的执行流
Execution exe = runtimeService.createExecutionQuery()
        .activityId("messageintermediatecatchevent1").singleResult();
// 触发消息事件
runtimeService.messageEventReceived("testMsg", exe.getId(), null);
// 查询执行流
exes = runtimeService.createExecutionQuery().processDefinitionKey("testProcess").list();
```

注：流程启动后就会到达消息事件节点，产生两条执行流记录。

注2：消息事件只能针对一个执行流，并不能像信号事件那样向全部执行流发生信号。

5）执行结果

在流程启动后添加断点，查看数据库的执行流记录和事件记录

![](/images/book/workflow/activiti/execution/catchingmessage1.png)

![](/images/book/workflow/activiti/execution/messageevent.png)

执行messageEventReceived方法后，查看执行流数据

![](/images/book/workflow/activiti/execution/catchingmessage2.png)