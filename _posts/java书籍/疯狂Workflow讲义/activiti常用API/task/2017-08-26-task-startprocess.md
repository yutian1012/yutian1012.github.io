---
title: 流程启动生成任务
tags: [activiti]
---

1）启动流程

```
ProcessDefinition pd = repositoryService.createProcessDefinitionQuery()
    .deploymentId(dep.getId()).singleResult();
// 启动流程，会生成Task任务对象
ProcessInstance pi = runtimeService.startProcessInstanceById(pd.getId());
```

2）查看RuntimeService的startProcessInstanceById方法

RuntimeService的实现类为RuntimeServiceImpl，查看该类的startProcessInstanceById方法。

```
public ProcessInstance startProcessInstanceById(String processDefinitionId) {
    return commandExecutor.execute(new StartProcessInstanceCmd<ProcessInstance>(null, processDefinitionId, null, null));
}
```

注：使用命令对象StartProcessInstanceCmd对象来封装了具体的执行过程

3）StartProcessInstanceCmd执行命令

```
ProcessDefinitionEntity processDefinition = null;
//获取ProcessDefinitionEntity对象，提供了2中方式获取

//根据processDefinitionId获取
if (processDefinitionId!=null) {
  processDefinition = deploymentCache.findDeployedProcessDefinitionById(processDefinitionId);
  if (processDefinition == null) {
    throw new ActivitiException("No process definition found for id = '" + processDefinitionId + "'");
  }
} 
//根据processDefinitionKey获取
else if(processDefinitionKey != null){
  processDefinition = deploymentCache.findDeployedLatestProcessDefinitionByKey(processDefinitionKey);
  if (processDefinition == null) {
    throw new ActivitiException("No process definition found for key '" + processDefinitionKey +"'");
  }
} else {
  throw new ActivitiException("processDefinitionKey and processDefinitionId are null");
}

//创建流程实例--重点
ExecutionEntity processInstance = processDefinition.createProcessInstance(businessKey);

//设置变量
if (variables!=null) {
  processInstance.setVariables(variables);
}

//启动流程实例
processInstance.start();

return processInstance;
```

4）ProcessDefinitionEntity类的createProcessInstance方法

createProcessInstance方法有很多重载。

```
//创建流程实例
public ExecutionEntity createProcessInstance(String businessKey) {
    return createProcessInstance(businessKey, null);
}

public ExecutionEntity createProcessInstance(String businessKey, ActivityImpl initial) {
    ExecutionEntity processInstance = null;
  
    if(initial == null) {
      processInstance = (ExecutionEntity) super.createProcessInstance();
    }else {
      processInstance = (ExecutionEntity) super.createProcessInstanceForInitial(initial);
    }

    CommandContext commandContext = Context.getCommandContext();
  
    processInstance.setExecutions(new ArrayList<ExecutionEntity>());
    processInstance.setProcessDefinition(processDefinition);
    // Do not initialize variable map (let it happen lazily)

    if (businessKey != null) {
        processInstance.setBusinessKey(businessKey);
    }
    
    // reset the process instance in order to have the db-generated process instance id available
    processInstance.setProcessInstance(processInstance);
    
    String initiatorVariableName = (String) getProperty(BpmnParse.PROPERTYNAME_INITIATOR_VARIABLE_NAME);
    if (initiatorVariableName!=null) {
      String authenticatedUserId = Authentication.getAuthenticatedUserId();
      processInstance.setVariable(initiatorVariableName, authenticatedUserId);
    }
    
    int historyLevel = Context.getProcessEngineConfiguration().getHistoryLevel();
    //设置ACT_HI_PROCINST表，流程的历史记录
    if (historyLevel>=ProcessEngineConfigurationImpl.HISTORYLEVEL_ACTIVITY) {
      HistoricProcessInstanceEntity historicProcessInstance = new HistoricProcessInstanceEntity(processInstance);

      commandContext
        .getSession(DbSqlSession.class)
        .insert(historicProcessInstance);

      //构造对象，保存流程活动的历史记录，act_hi_actinst表，其中开始节点的ACT_TYPE_字段为StartEvent，用户节点的ACT_TYPE_字段为userTask
      HistoricActivityInstanceEntity historicActivityInstance = new HistoricActivityInstanceEntity();
      historicActivityInstance.setId(idGenerator.getNextId());
      historicActivityInstance.setProcessDefinitionId(processDefinitionId);
      historicActivityInstance.setProcessInstanceId(processInstanceId);
      historicActivityInstance.setExecutionId(executionId);
      historicActivityInstance.setActivityId(processInstance.getActivityId());
      historicActivityInstance.setActivityName((String) processInstance.getActivity().getProperty("name"));
      historicActivityInstance.setActivityType((String) processInstance.getActivity().getProperty("type"));
      Date now = ClockUtil.getCurrentTime();
      historicActivityInstance.setStartTime(now);
      
      commandContext
        .getDbSqlSession()
        .insert(historicActivityInstance);
    }
```