---
title: 流程监听
tags: [bpmx]
---

1）设置监听

对ACTIVITI扩展，主要是扩展了流程的监听器，自动任务节点等。ACTIVITI流程定义为一个xml文件，在该xml中定义了相关监听器。

```
<process id="zlxmbglc" name="著录项目变更流程">
    <extensionElements>
        <activiti:executionListener event="start" 
            delegateExpression="${startEventListener}"/>
        <activiti:executionListener event="end" 
            delegateExpression="${endEventListener}"/>
    </extensionElements>
    <startEvent activiti:initiator="startUser" id="StartEvent1" name="开始"/>
    <endEvent id="EndEvent1" name="结束1"/>
    <userTask id="UserTask1" name="发明人">
        <documentation/>
        <extensionElements>
            <activiti:taskListener event="create" 
                delegateExpression="${taskCreateListener}"/>
            <activiti:taskListener event="assignment" 
                delegateExpression="${taskAssignListener}"/>
            <activiti:taskListener event="complete"
                delegateExpression="${taskCompleteListener}"/>
            <ht:order>1</ht:order>
        </extensionElements>
    </userTask>
    ...
</process>
```

注：在流程定义xml中通过extensionElements设置监听，该元素位于process标签下表示对流程设置监听，放置在userTask标签下表示对任务进行监听。

2）监听的执行时机

流程启动时，会触发流程启动事件监听器StartEventListener。

如果第一个节点为任务节点，那么将会调用TaskCreateListener监听器。流程引擎产生用户任务；

在流程表单中点击，执行下一步的按钮。这个时候会触发TaskCompleteListener监听器。如果下个节点是任务节点，则会触发TaskCreateListener监听器。

![](/images/work/bpmx/source/listener.png)

3）设置监听的执行

在流程定义--流程设置--节点设置上设置相应的事件（起始、结束阶段事件，任务事件等）。

如：设置流程启动的事件监听StartEventListener

![](/images/work/bpmx/source/setlistener.png)