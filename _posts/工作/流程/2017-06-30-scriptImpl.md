---
title: 流程中的脚本引擎scriptImpl
tags: [bpmx]
---

实例：为流程设置人员选择脚本（下一个节点执行人员），即在流程节点下设置流程执行人员。

### 定义

1）接口

实现自定义接口IScript，仅为一个标识接口，实现类可以直接被注入到脚本执行引擎，进行计算。

```
public interface IScript {
}
```

2）配置

```
<bean id="scriptImpl" class="com.hotent.platform.service.bpm.impl.ScriptImpl"></bean>
```

注：配置文件app-beans.xml

3）页面添加脚本

```
http://localhost:8080/eip-saas/platform/system/script/list.ht
```

通过编辑已存在的脚本可以发现实际使用的是scriptImpl实现类的某个方法

```
return scriptImpl.getUserIdsByOrgId(String orgId);
```

注：在执行时，会将调用参数orgId通过流程变量传递过去。

### 流程脚本的执行过程

1）流程设置脚本

一般在流程定义--设置--人员--脚本选择。

![](/images/work/bpmx/flowscript.png)

2）流程执行同意

流程执行同意操作，调用taskController类的complete方法，进而调用processRunService类的nextProcess，传递ProcessCmd作为封装参数信息。

ProcessCmd对象中包含流程定义信息，业务表单数据，流程变量集合等于流程相关的上下文信息。

nextProcess方法中调用completeTask方法，会使用taskService（activiti）类的complete方法，然后转向了流程设置脚本。

3）查看流程定义

在流程定义管理页面，点击bpmn-xml按钮，查看activiti流程定义xml信息

节点3--UserTask3中设置了执行脚本

```
<userTask id="UserTask3" name="总公司管理员">
    <documentation/>
    <extensionElements>
        <activiti:taskListener event="create" delegateExpression="${taskCreateListener}"/>
        <activiti:taskListener event="assignment" delegateExpression="${taskAssignListener}"/>
        <activiti:taskListener event="complete" delegateExpression="${taskCompleteListener}"/>
        <ht:order>3</ht:order>
    </extensionElements>
</userTask>
```

注：这里定义了3个监听，分别是任务创建（taskCreateListener），任务分配（taskAssignListener），认为完成（taskCompleteListener）监听器。

4）监听

分别查看这三个监听的实现，如TaskCreateListener类

```
com.hotent.platform.service.bpm.listener.TaskCreateListener
```

该类实现了org.activiti.engine.delegate.TaskListener接口。可在activiti流程中被回调。

5）查看TaskCreateListener类

由于设置的流程脚本是用于流程节点人员设置，因此，主要查看该监听的实现。首先TaskListener接口中存在一个notify方法，供子类实现。即TaskCreateListener类需要实现该接口方法，具体实现可查看TaskCreateListener类的基类BaseTaskListener。

查看BaseTaskListener类的notify方法

```
@Override
public void notify(DelegateTask delegateTask) {
    
    TaskEntity taskEnt=(TaskEntity)delegateTask;//Task任务实体
    String nodeId=taskEnt.getExecution().getActivityId();//获取执行的环节ID
    String actDefId=taskEnt.getProcessDefinitionId();//获取流程定义ID
    
    //获取脚本类型
    int beforeScriptType=getBeforeScriptType();
    //执行前置事件脚本
    exeEventScript(delegateTask,beforeScriptType,actDefId,nodeId);
    
    logger.debug("enter the baseTaskListener notify method...");
    
    //执行子类业务逻辑--子类中实现
    execute(delegateTask,actDefId,nodeId);
    //获取脚本类型
    int scriptType=getScriptType();
    //执行后置事件脚本
    exeEventScript(delegateTask,scriptType,actDefId,nodeId);
    
}
```

通过生成任务，并创建出任务的候选执行人，进而继续流程的执行。

注：execute是一个模板方法，供具体类实现，并在基类中回调。

6）最终用户的获取（候选用户）

执行代码查看TaskCreateListener的execute方法（模板方法）中，该方法中会获取用户，将用户设置到候选用户中。execute会调用handAssignUserFromDb方法。

TaskCreateListener类的handAssignUserFromDb方法的实现

```
/**
 * 从数据库加载人员并分配用户。
 * @param actDefId
 * @param nodeId
 * @param delegateTask
 */
private void handAssignUserFromDb(String actDefId,String nodeId,DelegateTask delegateTask){
    //脚本配置后会存放到bpm_node_user表中，通过该service完成获取脚本的功能
    BpmNodeUserService userService=(BpmNodeUserService) AppUtil.getBean(BpmNodeUserService.class);
    
    String actInstId=delegateTask.getProcessInstanceId();//流程实例ID
    
    //获取流程实例
    ProcessInstance processInstance=bpmService.getProcessInstance(actInstId);
    List<TaskExecutor> users=null; 

    //获取流程变量。
    Map<String,Object> vars=delegateTask.getVariables();
    
    vars.put(BpmConst.EXECUTION_ID_, delegateTask.getExecutionId());
    //执行任务的情况
    if(processInstance!=null){
        //获取上个任务的执行人，这个执行人在上一个流程任务的完成事件中进行设置。
        //代码请参考TaskCompleteListener。
        String startUserId=(String)vars.get(BpmConst.StartUser);
        
        //上一步流程执行人，即当前用户执行的同意操作
        String preStepUserId=ContextUtil.getCurrentUserId().toString();
        Long preStepOrgId=ContextUtil.getCurrentOrgId();
        vars.put(BpmConst.PRE_ORG_ID, preStepOrgId);
        
        if(StringUtil.isEmpty(startUserId) && vars.containsKey(BpmConst.PROCESS_INNER_VARNAME)){
            Map<String,Object> localVars=(Map<String,Object>)vars.get(BpmConst.PROCESS_INNER_VARNAME);
            startUserId=(String)localVars.get(BpmConst.StartUser);
        }
        
        users=userService.getExeUserIds(actDefId, actInstId, nodeId, startUserId,preStepUserId,vars);
    }
    //启动流程
    else{
        //startUser
        //上个节点的任务执行人
        String startUserId=(String)vars.get(BpmConst.StartUser);
        //内部子流程启动
        if(StringUtil.isEmpty(startUserId) && vars.containsKey(BpmConst.PROCESS_INNER_VARNAME)){
            Map<String,Object> localVars=(Map<String,Object>)vars.get(BpmConst.PROCESS_INNER_VARNAME);
            startUserId=(String)localVars.get(BpmConst.StartUser);
        }
        users=userService.getExeUserIds(actDefId, actInstId, nodeId, startUserId, startUserId,vars);
    }
    assignUser(delegateTask,users);
}
```

6）根据指派人员类型获取人员计算实现类

人员计算使用策略模式

```
IBpmNodeUserCalculation calculation = bpmNodeUserCalculationSelector.getByKey(bpmNodeUser.getAssignType());
```

其中bpmNodeUserCalculationSelector类的配置信息位于app-beans.xml

```
<bean id="bpmNodeUserCalculationSelector"
    class="com.hotent.platform.service.bpm.BpmNodeUserCalculationSelector">
    <property name="bpmNodeUserCalculation">
        <map>
            ...
            <!-- 使用脚本 -->
            <entry key="script" value-ref="scriptCalculation" />
            ...
        </map>
    </property>
</bean>

<bean id="scriptCalculation"    class="com.hotent.platform.service.bpm.impl.BpmNodeUserCalculationScript"></bean>
```

这里设置的指派人员的类型为script（脚本类型），因此会交由BpmNodeUserCalculationScript类的getTaskExecutor方法来获取执行人员，最终由groovyScriptEngine.executeObject(script, grooVars)来执行。

7）脚本方法的参数如何设置

查看BpmNodeUserCalculationScript类的getTaskExecutor方法实现

```
public List<SysUser> getExecutor(BpmNodeUser bpmNodeUser, CalcVars vars) {
    Long prevUserId = vars.getPrevExecUserId();
    Long startUserId = vars.getStartUserId();
    
    String script = bpmNodeUser.getCmpNames();//脚本内容
    List<SysUser> users = new ArrayList<SysUser>();

    //执行参数设置
    Map<String, Object> grooVars = new HashMap<String, Object>();
    
    ProcessCmd cmd=TaskThreadService.getProcessCmd();
    
    String actInstId="";
    if(StringUtil.isNotEmpty(vars.getActInstId()) ){
        actInstId=vars.getActInstId();
        if (cmd!=null) {
            cmd.addTransientVar("actInstId", actInstId);
        }
    }
    //封装groovy使用的脚本参数
    grooVars.put("actInstId", actInstId);
    if(vars.getVars().size()>0){
        grooVars.putAll(vars.getVars());
    }
    grooVars.put(BpmConst.PrevUser, prevUserId);
    grooVars.put(BpmConst.StartUser, startUserId.toString());
    
    Object result = groovyScriptEngine.executeObject(script, grooVars);
    if (result == null) {
        return users;
    }
    Set<String> set = (Set<String>) result;
    for (Iterator<String> it = set.iterator(); it.hasNext();) {
        String userId = it.next();
        SysUser sysUser = sysUserService.getById(Long.parseLong(userId));
        users.add(sysUser);
    }
    return users;
}
```

注：CalcVars中包含流程变量，流程定义等信息。

8）groovy引擎执行

配置文件app-bean.xml

```
<!-- 脚本引擎 -->
<bean id="scriptEngine" class="com.hotent.core.engine.GroovyScriptEngine"></bean>
```

```
public Object executeObject(String script,Map<String, Object> vars) {
    GroovyShell shell = new GroovyShell(binding);
    setParameters(shell,vars);//设置参数shell.setVariable完成
    
    //放置脚本注入攻击
    script=script.replace("&apos;", "'").replace("&quot;", "\"").replace("&gt;", ">").replace("&lt;", "<").replace("&nuot;", "\n").replace("&amp;", "&");
    
    //执行脚本并获取对象
    Object rtn =  shell.evaluate(script);
    //清空绑定变量
    binding.clearVariables();
    return rtn;
}
```

binding对象的定义

```
GroovyBinding binding = new GroovyBinding();
```

### 脚本类型

