---
title: 流程实例结束监听源码
tags: [bpmx]
---

EndEventListener流程监听继承org.activiti.engine.delegate.ExecutionListener类。主要实现抄送，流程终止（手动终止），更新业务中间表数据状态，处理发送提醒消息给发起人，更新流程运行状态，流程终止时删除流程任务，以及在结束节点上设置执行脚本（groovy）。

1）继承体系

EndEventListener-->BaseNodeEventListener-->ExecutionListener

2）ExecutionListener的执行时机

查看activiti的源码：

TaskService类的complete方法，使用命令对象CompleteTaskCmd执行。

CompleteTaskCmd类的execute方法中调用TaskEntity类的complete方法。该方法会调用execution.signal方法（即ExecutionEntity类的signal方法）。

```
//会获取处理流程结束--UserTaskActivitiBehavior
SignallableActivityBehavior activityBehavior = (SignallableActivityBehavior) activity.getActivityBehavior();

activityBehavior.signal(this, signalName, signalData);
```

UserTaskActivitiBehavior类的signal方法会调用父类AbstractBpmnActivityBehavior的leave方法，进而再调用上层父类FlowNodeActivityBehavior的leave方法。

FlowNodeActivityBehavior类的leave方法

```
//获取流程节点出去的连线
protected void leave(ActivityExecution execution) {//传递执行流
    bpmnActivityBehavior.performDefaultOutgoingBehavior(execution);
}
```

注：内部经过一系列的流转后会调用execution执行流的take方法。

ExecutionEntity类的take方法

```
public void take(PvmTransition transition) {
    if (this.transition!=null) {
      throw new PvmException("already taking a transition");
    }
    if (transition==null) {
      throw new PvmException("transition is null");
    }
    setActivity((ActivityImpl)transition.getSource());
    setTransition((TransitionImpl) transition);
    //AtomicOperationTransitionNotifyListenerEnd执行
    performOperation(AtomicOperation.TRANSITION_NOTIFY_LISTENER_END);
}
```

performOperation会不断的被调用，执行不同的原子操作。

```
AtomicOperationTransitionNotifyListenerEnd
AtomicOperationTransitionCreateScope
AtomicOperationTransitionNotifyListenerTake
AtomicOperationTransitionCreateScope
AtomicOperationTransitionNotifyListenerStart
AtomicOperationActivityExecute
```

AtomicOperationActivityExecute类会调用ExecutionEntity类的end方法，end方法又会调用相应的原子操作

```
AtomicOperationActivityEnd
AtomicOperationProcessEnd
```

AtomicOperationProcessEnd类流程结束事件的监听，并执行。查看父类AbstractEventAtomicOperation的execute方法。

```
List<ExecutionListener> exectionListeners = scope.getExecutionListeners(getEventName());
int executionListenerIndex = execution.getExecutionListenerIndex();

if (exectionListeners.size()>executionListenerIndex) {
  execution.setEventName(getEventName());
  execution.setEventSource(scope);
  ExecutionListener listener = exectionListeners.get(executionListenerIndex);
  try {
    listener.notify(execution);
  } catch (RuntimeException e) {
    throw e;
  } catch (Exception e) {
    throw new PvmException("couldn't execute event listener : "+e.getMessage(), e);
  }
  execution.setExecutionListenerIndex(executionListenerIndex+1);
  execution.performOperation(this);
}
```

注：最终调到了结束事件的监听

3）BaseNodeEventListener的执行

该类实际上是一个模板方法，在notify方法中定义了执行的顺序，并声明了抽象方法execute和getScriptType。

4）EndEventListener的执行

该类重新模板类中定义的两个抽象方法execute和getScriptType方法。并提供了相关功能的实现（如：抄送，流程终止，处理发送提醒消息给发起人等）。