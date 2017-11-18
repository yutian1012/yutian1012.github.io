---
title: 流程节点中人员的策略模式
tags: [bpmx]
---

在app-beans.xml中配置了相应的策略实现类，在实际使用时根据设置的bpmNodeUser对象取得，获取用户所使用的策略方法，从而计算出实际的用户集合。

1）查看app-beans.xml中的配置项bpmNodeUserCalculationSelector

该类是一个策略模式的Context（上下文环境），内部包含一个map集合，用来存储相应的策略实现类，每一个key对应唯一的一个策略。

```
private Map<String,IBpmNodeUserCalculation > bpmNodeUserCalculation=new LinkedHashMap<String, IBpmNodeUserCalculation>();
```

并提供了相应的get和set方法，用来获取和设置相关策略。

2）IBpmNodeUserCalculation接口

该类中包含一个内部类PreViewModel，该类用来设置查看预览模型对象，包含预览模型类型以及标题信息。

核心方法

```
/**
 * 用于预览。
 * @param bpmNodeUser
 * @param vars
 * @return
 */
List<SysUser> getExecutor(BpmNodeUser bpmNodeUser,CalcVars vars);

/**
 * 用于流程任务。
 * @param bpmNodeUser
 * @param vars
 * @return
 */
Set<TaskExecutor> getTaskExecutor(BpmNodeUser bpmNodeUser,CalcVars vars);
```

注：前者是用来进行预览时显示的人员信息。获取的是user集合，后者是流程任务执行人员，根据是否抽取的设置，获取用户或者角色，组织等信息，然后通过设置获取角色或组织中的人员作为流程执行人。

3）实现类

根据该策略接口，系统提供了十几个具体的策略实现类

```
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
...
```

