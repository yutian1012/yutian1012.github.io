---
title: 流程url表单配置流程变量
tags: [bpmx]
---

1）流程变量的作用

有时候流程的跳转，与节点的执行人都是在表单中指定的，这时就需要能够获取表单的输入数据。进而实现流程的转向和执行人的处理。

对于在线流程表单的处理，在定义表时勾选流程变量，会自动设置到流程中。然而，如果使用url表单，流程变量又如何获取呢。

2）设置流程变量

对于url表单，没有办法直接利用已有的功能实现流程表单的定义。可以使用如下方式获取表单中的数据。

方式一：执行sql语句的方式获取保存到数据库中的值

```
String director=scriptImpl.executeSql("select director_id from whlg_patent_apply_flow where id="+businessKey);
```

方式二：通过定义相应的脚步方法获取

在ScriptImpl类中定义相关的方法，并添加到脚步设置中，便可以选择数据

```
public Set<String> getSetByChargId(Long chargeId){
    ...
}
```

方式三：在前置处理器中处理数据

可事先在流程管理--流程定义管理--变量管理中添加变量，目的是为了能够在人员设置和跳转设置时获取该变量。具体变量的值无法直接设置上，但是可以通过前置处理器进行设置。

```
//设置流程变量
cmd.getVariables().put("directorId", whlgPatentApplyFlow.getDirectorId());
```

注：可直接调用ProcessCmd对象的getVariables方法获取变量集合，然后向map中设置变量值。