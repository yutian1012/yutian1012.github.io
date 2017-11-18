---
title: 流程节点中人员的计算
tags: [bpmx]
---

用户人员的计算是在任务节点创建监听中计算的，查看TaskCreateListener，当没有设置任务执行人时，会调用handAssignUserFromDb方法，即根据节点的设置信息获取流程节点的候选人。

1）从数据库中加载人员并分配用户

```
private void handAssignUserFromDb(String actDefId,String nodeId,DelegateTask delegateTask){
    List<TaskExecutor> users=null; 
    ...
    //将流程启动人，上一节点执行人，流程变量，节点ID等信息作为参数查询
    users=userService.getExeUserIds(actDefId, actInstId, nodeId, startUserId,preStepUserId,vars);
    ...
}
```

注：userService是BpmNodeUserService类的实例

2）BpmNodeUserService类的getExeUserIds方法

```
public List getExeUserIds(String actDefId, String actInstId, String nodeId, String startUserId,String preTaskUser,Map<String,Object> vars) {
    ...
    BpmNodeSet bpmNodeSet = null;
    ...
    //获取节点配置信息，bpm_node_set表信息，主要针对表单设置信息
    bpmNodeSet = bpmNodeSetDao.getByActDefIdNodeId(actDefId, nodeId);
    //如果任务节点上未设置表单信息，则直接返回空集合
    if(bpmNodeSet==null) return list;
    //流程设置--人员，在节点上设置人员调解，和人员策略（信息存储bpmUserCondition中）
    //根据setId获取设置的人员条件列表。
    List<BpmUserCondition> bpmUserConditionList = bpmUserConditionDao.getBySetId(bpmNodeSet.getSetId());
    //通过解析人员条件获取相应的人员信息
    list=getExecutorsByConditions(bpmUserConditionList, actDefId, actInstId, startUserId, preTaskUser, vars,false);
}
```

3）BpmNodeUserService类的getExecutorsByConditions方法。

根据条件列表，流程定义ID,流程实例ID,发起人ID,上个执行人ID,流程变量,是否取得所有的用户。

```
private List getExecutorsByConditions(List<BpmUserCondition> bpmUserConditionList,String actDefId,String actInstId,String startUserId,String preTaskUser,Map<String,Object> vars,boolean isUser){
    ...
    //是否找到了执行人
    boolean hasExecutor=false;
    
    BpmUserCondition prevCondition=null;
    //循环变量conditions集合，每个节点的人员都存在批次信息
    for(int i=0;i<bpmUserConditionList.size();i++){
        BpmUserCondition currentCondition = bpmUserConditionList.get(i);
        Long conditionId=currentCondition.getId();
        //上一个规则不为空 ,并且前面的批次和当前的批次不一样，且之前的规则找到用户了，这个时候跳出循环。
        if(prevCondition!=null && !prevCondition.getGroupNo().equals(currentCondition.getGroupNo()) && hasExecutor ){
            break;
        }
        //检查条件，校验设置的规则信息，
        boolean isPass = conditionCheck(currentCondition, formIdentity, vars);
        if(!isPass) continue;
        //取得执行人
        List partUsers=getExecutors(conditionId, actDefId, actInstId,  startUserId, preTaskUser,vars,isUser);
        if(partUsers.size()>0){
            userSet.addAll(partUsers);
            hasExecutor=true;
        }
        prevCondition = currentCondition;
    }
    list.addAll(userSet);
    return list;
}
```

4）获取用户集合getExecutors方法

```
public List getExecutors(Long conditionId,String actDefId, String actInstId,  String startUserId,String preTaskUser,Map<String,Object> vars,boolean isUser){
    List<BpmNodeUser> nodeUsers = bpmNodeUserDao.getByConditionId(conditionId);
    List list=new ArrayList();
    Set userIdSet = new HashSet();
    
    for (BpmNodeUser bpmNodeUser : nodeUsers) {
        Set uIdSet=null;
        if(isUser){//直接获取用户信息
            uIdSet=getUsersByBpmNodeUser( bpmNodeUser, startUserId, preTaskUser, actInstId,vars);
        }
        else{//不直接获取用户，而根据设置的是否抽取来决定是否获取用户
            uIdSet=getByBpmNodeUser( bpmNodeUser, startUserId, preTaskUser, actInstId,vars);
        }
        if (userIdSet.size() == 0) {
            userIdSet = uIdSet;
        }
        else {
            //条件组合，交，差，并的运算
            userIdSet = computeUserSet(bpmNodeUser.getCompType(), userIdSet, uIdSet);
        }
    }
    list.addAll(userIdSet);
    return list;
}
```

注：isUser是否直接抽取用户，这个在抄送，消息节点人员获取时使用.

5）bpmNodeUserService类的getByBpmNodeUser方法

```
private Set<TaskExecutor> getByBpmNodeUser(BpmNodeUser bpmNodeUser,String startUserId,String preTaskUserId,String actInstId,Map<String,Object> vars){       
    ...
    //根据bpmNodeUser信息获取相应的策略实现类，然后调用该类的getTaskExecutor获取对应的用户集合。
    CalcVars params= new CalcVars(lStartUserId,lPrevTaskUserId,actInstId,vars);
    IBpmNodeUserCalculation calculation = bpmNodeUserCalculationSelector.getByKey(bpmNodeUser.getAssignType());
    return calculation.getTaskExecutor(bpmNodeUser, params);
}
```