---
title: 我的草稿启动流程
tags: [bpmx]
---

我的草稿列表中，选择一条记录启动流程（管理列的启动流程按钮）。

1）组件

我的草稿页面：processRunMyForm.jsp

处理类：ProcessRunController

草稿数据保存位置：

2）启动流程检查草稿表单是否发生变化

ProcessRunController类的checkForm方法。