---
title: 流程表单页面渲染
tags: [js]
---

1、表单中的复选框和单选框的选中事件

在流程自定义表单时，经常会使用复选框等控件，在自定义表设置字段时，也可以指定选项。运行表单选中复选框之后如何回显？

流程表单初始化js

```
<script type="text/javascript" src="${ctx}/js/hotent/platform/form/CustomForm.js"></script>
```

CustomForm.js中的initUI方法，用来初始化控件值的设置信息。

1）复选框值的绑定

```
<input name="m:tProposal:proposalType" value="项目类" type="radio" data="${tProposal.proposalType}" />项目类
```

注：这里的data属性表示了用户选中的值

2）具体CustomForm.js中的处理代码

```
$('input[type="checkbox"],input[type="radio"]').each(function() {
    var value=$(this).val();
    var data=$(this).attr('data');
    if(!data||data==''||data=='null')return;
    var ary=data.split(",");
    for(var i=0;i<ary.length;i++){
        if(value!=ary[i]) continue;
        $(this).attr("checked","checked");
    }
});
```