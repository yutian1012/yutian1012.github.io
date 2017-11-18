---
title: 流程任务结束监听源码
tags: [bpmx]
---

TaskCompleteListener任务监听继承了org.activiti.engine.delegate.TaskListener。任务监听可以分别设置前置和后置脚本。处理任务分发令牌，更新执行堆栈里的执行人员及完成时间，更新任务意见，更新流程节点状态，更新历史节点，删除任务已读记录等。

注：流程监听继承的是ExecutionListener类。

1）继承体系

TaskCompleteListener-->BaseTaskListener-->TaskListener

2）TaskListener的执行时机

查看activiti的源码：

TaskService类的complete方法，使用命令对象CompleteTaskCmd执行。

CompleteTaskCmd类的execute方法中调用TaskEntity类的complete方法。

```
//在获取监听时，会根据此名称获取完成的监听，而不会获取到创建的监听
fireEvent(TaskListener.EVENTNAME_COMPLETE);
```

注：TaskEntity的complete方法会触发事件TaskListener.EVENTNAME_COMPLETE

TaskEntity类的fireEvent方法

```
public void fireEvent(String taskEventName) {
    //获取task节点的定义信息（流程定义文件xml）
    TaskDefinition taskDefinition = getTaskDefinition();
    if (taskDefinition != null) {
      //获取task节点上的监听，
      //这里传递的name为complete，因此获取的监听是完成监听
      List<TaskListener> taskEventListeners = getTaskDefinition().getTaskListener(taskEventName);
      if (taskEventListeners != null) {
        for (TaskListener taskListener : taskEventListeners) {
          ExecutionEntity execution = getExecution();
          if (execution != null) {
            setEventName(taskEventName);
          }
          try {
            //调用监听执行
            Context.getProcessEngineConfiguration()
              .getDelegateInterceptor()
              .handleInvocation(new TaskListenerInvocation(taskListener, (DelegateTask)this));
          }catch (Exception e) {
            throw new ActivitiException("Exception while invoking TaskListener: "+e.getMessage(), e);
          }
        }
      }
    }
}
```

注：Context.getProcessEngineConfiguration().getDelegateInterceptor()返回DefaultDelegateInterceptor对象，该类的handleInvocation方法会调用实参的invoke方法（即调用TaskListenerInvocation类的invoke方法）。

```
public class TaskListenerInvocation extends DelegateInvocation {

  protected final TaskListener executionListenerInstance;
  protected final DelegateTask delegateTask;

  public TaskListenerInvocation(TaskListener executionListenerInstance, DelegateTask delegateTask) {
    this.executionListenerInstance = executionListenerInstance;
    this.delegateTask = delegateTask;
  }

  protected void invoke() throws Exception {
    executionListenerInstance.notify(delegateTask);
  }
  
  public Object getTarget() {
    return executionListenerInstance;
  }
}
```

注：invoke方法中执行监听的notify方法，即任务监听得到了调用。

3）BaseTaskListener

该类实际上是一个模板方法，在notify方法中定义了执行的顺序，并声明了抽象方法execute，getBeforeScriptType和getScriptType等方法。

```
public void notify(DelegateTask delegateTask) {
    TaskEntity taskEnt=(TaskEntity)delegateTask;
    String nodeId=taskEnt.getExecution().getActivityId();//获取执行的环节ID
    String actDefId=taskEnt.getProcessDefinitionId();//获取流程定义ID
    
    //获取前置脚本类型
    int beforeScriptType=getBeforeScriptType();
    //执行前置事件脚本
    exeEventScript(delegateTask,beforeScriptType,actDefId,nodeId);
    
    logger.debug("enter the baseTaskListener notify method...");
    
    //执行子类业务逻辑--子类中实现
    execute(delegateTask,actDefId,nodeId);
    
    //获取后置脚本类型
    int scriptType=getScriptType();
    //执行后置事件脚本
    exeEventScript(delegateTask,scriptType,actDefId,nodeId);
}
```

4）TaskCompleteListener

该类重写模板类中定义的抽象方法，execute中主要实现了处理任务分发令牌，更新执行堆栈里的执行人员及完成时间，更新任务意见，更新流程节点状态，更新历史节点，删除任务已读记录等。