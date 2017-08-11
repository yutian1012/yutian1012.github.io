---
title: 流程中与流程流转相关的常量信息
tags: [bpmx]
---

在流程的执行过程中，存在很多的常量值，包括按钮（BpmNodeButton类中定义的常量），意见（TaskOpinion类中定义的常量），流程状态（ProcessRun类中定义的常量），页面封装的状态（ProcessCmd类的voteAgree）等。

问题：这些常量间有什么关系？

1）流程任务状态

流程任务状态在数据库中是以Short类型记录，每一种数值表示的状态在实体类TaskOpinion中也以静态变量的方式列出。查看bpm_task_opinion表的checkstatus字段。

2）流程实例状态

在流程运行过程中流程实例状态，在ProcessRun的静态变量中列出。