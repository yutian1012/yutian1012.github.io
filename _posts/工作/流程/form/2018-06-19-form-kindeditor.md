---
title: 表单富文本框使用
tags: [bpmx]
---

1）页面中引入

```
<%@include file="/commons/include/kindeditor.jsp" %>
```

2）在textarea上添加class属性

```
<textarea class="myeditor" style="width:90%" type="text" 
    id="patentAbs" name="patentAbs" validate="{required:true}">
    ${whlgPatentApplyFlow.patentAbs}
</textarea>
```

3）设置富文本框内容提交

```
$("textarea[name^='m:'].myeditor").each(function(num) {
    var name=$(this).attr("name");
    var data=getEditorData(editor[num]);                    
    $("textarea[name^='"+name+"']").val(data);
});
```