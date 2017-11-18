---
title: 流程节点中人员的选择
tags: [bpmx]
---

1）配置文件

在app-beans.xml中配置了流程节点人员设置方式，如发起人，指定用户，指定角色等方式设置节点执行人。

```
<bean id="bpmNodeUserCalculationSelector"
    class="com.hotent.platform.service.bpm.BpmNodeUserCalculationSelector">
    <property name="bpmNodeUserCalculation">
        <map>
            <!-- 发起人 -->
            <entry key="startUser" value-ref="startUserCalculation" />
            <!-- 指定用户 -->
            <entry key="users" value-ref="userCalculation" />
            <!-- 指定角色 -->
            <entry key="role" value-ref="roleCalculation" />
            <!-- 指定组织 -->
            <entry key="org" value-ref="orgCalculation" />
            <!-- 指定组织负责人 -->
            <entry key="orgCharge" value-ref="orgChargeCalculation" />
            <!-- 指定岗位 -->
            <entry key="pos" value-ref="positionCalculation" />
            <!-- 指定职务 <entry key="job" value-ref="jobCalculation" /> -->
            <!-- 指定上下级 -->
            <entry key="upLow" value-ref="upLowCalculation" />
            <!-- 用户属性 -->
            <entry key="userAttr" value-ref="userAttrCalculation" />
            <!-- 组织属性 -->
            <entry key="orgAttr" value-ref="orgAttrCalculation" />
            <!-- 与已执行节点相同执行人 -->
            <entry key="sameNode" value-ref="sameNodeCalculation" />
            <!-- 使用脚本 -->
            <entry key="script" value-ref="scriptCalculation" />
            <!-- 部门的上级类型部门的负责人 -->
            <entry key="orgTypeUserLeader" value-ref="pervTypeUserLeaderCalculation" />
            <!-- 表单变量 -->
            <entry key="formVar" value-ref="formVarCalculation" />
            <!-- 发起人或上一个任务执行人 -->
            <entry key="startOrPrev" value-ref="startOrPrevCalculation" />
            <!-- 发起人或上一个任务执行人的组织 -->
            <entry key="startOrPrevWithOrg" value-ref="startOrPrevWithOrgCalculation" />
            <!-- 审批过的人员 -->
            <entry key="approve" value-ref="approveCalculation" />
            <!-- 与已执行节点执行人相同部门 -->
            <entry key="sameNodeDepartment" value-ref="sameNodeCalculationDepartment" />
            <!-- 人员脚本 -->
            <entry key="personScript" value-ref="personScriptCalculationAssignUsers" />
            <!-- 发起节点配置节点人员
            <entry key="assignUsers" value-ref="nodeUserCalculationAssignUsers" /> -->
        </map>
    </property>
</bean>
```

注：这里使用的是一种策略模式。

2）配置流程节点人员的操作

流程管理--流程定义管理--设置--人员，在相应的节点下添加选择人员策略。

BpmDefinitionController类的conditionEdit（设置人员对话框），该方法主要操作是获取流程定义信息，流程变量信息，bpmUserCondition（人员选择条件设置信息的封装），BpmNodeUser（人员选择策略的封装）等。

conditionEdit方法

```
public ModelAndView conditionEdit(HttpServletRequest request,HttpServletResponse response) throws Exception {
    ...
    //流程定义信息
    BpmDefinition bpmDefinition = bpmDefinitionService.getById(defId);
    ...
    //获取人员条件信息，bpm_user_condition表，记录了节点等信息
    bpmUserCondition= bpmUserConditionService.getById(conditionId);
    ...
    //节点人员设置，bpm_node_user表，其中assignType值为人员策略map集合的key
    bpmNodeUserService.getByConditionId(conditionId);
    ...
    //获取人员类型。人员策略
    Map<String, IBpmNodeUserCalculation> assignUserTypes = bpmNodeUserCalculationSelector.getBpmNodeUserCalculation();
    ...
}
```

跳转页面bpmDefinitionConditionEdit.jsp

3）bpm_user_condition与bpm_node_user表关系

bpm_user_condition表的ID与bpm_node_user表的conditionId字段对应，bpm_node_user表中存储的是人员选择策略信息，以及相关的参数值，属于一对多的关系。

bpm_user_condition表的setId与bpm_node_set表的SetId对应，bpm_node_set表用于存储流程节点表单的设置信息，如果节点没有设置表单，则无法在节点上设置执行人。存在一个先后的关系。

另外，bpm_user_condition的conditionCode字段，存储了相应的人员设置规则信息，只有满足规则条件，才能继续使用相关的策略获取人员。

4）人员策略设置对话框

查看bpmDefinitionConditionEdit.jsp页面中引入的人员设置模板

```
<%@include file="/commons/include/nodeUserTemplate.jsp" %>
```

相应的对话框的设置位于js文件中，该js文件会根据不同的assignType，弹出不同的设置功能。

```
${ctx}/js/hotent/platform/form/nodeUserSelectorJS.js
```

