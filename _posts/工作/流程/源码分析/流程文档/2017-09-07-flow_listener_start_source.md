---
title: 流程实例启动监听源码
tags: [bpmx]
---

StartEventListener流程监听继承org.activiti.engine.delegate.ExecutionListener类。主要是解决子流程启动初始化问题，以及在启动节点上设置执行脚本（groovy）。

1）类的继承体系

StartEventListener-->BaseNodeEventListener-->ExecutionListener

2）ExecutionListener的执行时机

查看activiti的源码：

RuntimeServiceImpl类的startProcessInstanceByKey方法，使用命令对象StartProcessInstanceCmd执行。

StartProcessInstanceCmd类的execute方法，调用ExecutionEntity（即ProcessInstance）的start方法。start方法又会调用内部的performOperation方法。

ExecutionEntity内部的performOperation方法

```
public void start() {
    if(startingExecution == null && isProcessInstanceType()) {
      startingExecution = new StartingExecution(processDefinition.getInitial());
    }
    performOperation(AtomicOperation.PROCESS_START);
}

public void performOperation(AtomicOperation executionOperation) {
    if(executionOperation.isAsync(this)) {
      scheduleAtomicOperationAsync(executionOperation);
    } else {
      performOperationSync(executionOperation);
    }    
}

protected void performOperationSync(AtomicOperation executionOperation) {
    Context.getCommandContext()
      .performOperation(executionOperation, this);
}
```

CommandContext类的performOperation方法，传递过来的AtomicOperation真实对象为AtomicOperation.PROCESS_START。该方法会调用AtomicOperation.PROCESS_START类的execute方法。

AtomicOperationProcessStart类的execute方法，其中execute方法位于子类AbstractEventAtomicOperation中

```
public void execute(InterpretableExecution execution) {
    ScopeImpl scope = getScope(execution);
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

    } else {
      execution.setExecutionListenerIndex(0);
      execution.setEventName(null);
      execution.setEventSource(null);
      
      eventNotificationsCompleted(execution);
    }
}
```

注：可以看到该法方中获取监听，并调用监听的notify方法。这也是流程启动监听被调用的地方。

3）BaseNodeEventListener的执行

该类实际上是一个模板方法，在notify方法中定义了执行的顺序，并声明了抽象方法execute和getScriptType。

模板方法notify

```
public void notify(DelegateExecution execution) throws Exception {
    logger.debug("enter the node event listener.." + execution.getId());
    
    //强转成ExecutionEntity执行流对象
    ExecutionEntity ent=(ExecutionEntity)execution;

    String actDefId=ent.getProcessDefinitionId();
    //节点的id信息，如usertask1，流程节点的名称
    String nodeId=ent.getActivityId();
    
    //执行实现类业务逻辑
    execute(execution,actDefId,nodeId);
    //只有节不为空时，才会去执行脚本。
    if(nodeId!=null){
        //获取事件类型
        Integer scriptType=getScriptType();
        //执行事件脚本--在页面中设置的脚本内容
        exeEventScript(execution,scriptType,actDefId,nodeId);
    }
}
```

脚本执行方法exeEventScript

```
private void exeEventScript(DelegateExecution execution,int scriptType,
    String actDefId,String nodeId ){
    BpmNodeScriptService bpmNodeScriptService=(BpmNodeScriptService)AppUtil.getBean("bpmNodeScriptService");
    //获取设置的脚本内容
    BpmNodeScript model=bpmNodeScriptService.getScriptByType(nodeId, actDefId, scriptType);
    
    if(model==null) return;

    String script=model.getScript();
    if(StringUtil.isEmpty(script)) return;
    
    //使用Groovy脚本引擎加载并执行脚本
    GroovyScriptEngine scriptEngine=(GroovyScriptEngine)AppUtil.getBean("scriptEngine");
    //获取流程变量
    Map<String, Object> vars=execution.getVariables();
    vars.put("execution", execution);
    
    //执行脚本
    scriptEngine.execute(script, vars);
    
    logger.debug("execution script :" + script);
}
```

4）StartEventListener类

该类重新模板类中定义的两个抽象方法execute和getScriptType方法。其中execute方法主要处理子流程相关的业务，getScriptType返回BpmConst.StartScript。