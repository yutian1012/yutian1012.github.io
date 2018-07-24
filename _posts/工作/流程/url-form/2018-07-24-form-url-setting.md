---
title: 流程url表单配置
tags: [bpmx]
---

参考帮助文档设置url表单

1）通过代码生成器生成源代码

URL表单一般是通过代码生成器生成的表单或其他有接口的系统的表单。即表单是由自己手动创建的。

注：下面需要将表单设置到流程节点管理中。

2）创建流程

在流程节点上配置url表单

全局设置：

选择全局表单：/whlg/test/testUrlForm/modify.ht?id={pk}

前置处理器：testUrlFormService.modify

流程实例业务表单：

表单明细URL：/whlg/test/testUrlForm/detail.ht?id={pk}

节点表单：

url表单：/whlg/test/testUrlForm/modify.ht?id={pk}

前置处理器：testUrlFormService.modify

3）{pk}的理解

${pk}是何时被替换的？

启动流程，执行taskController类的startFlowForm方法，获取FormModel的方法（getStartForm）中会替换掉${pk}

代办流程，执行taskController类的toStart方法，通过taskId获取相关节点信息，并提供${pk}

4）控制器

在相应的controller类中有modify方法，用于跳转的相应的表单。