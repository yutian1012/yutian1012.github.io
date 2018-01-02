---
title: easy-ui中查询工具条
tags: [easy-ui]
---

参考：http://www.jeasyui.com/documentation/index.php

### 自定义查询工具栏

自定义查询工具栏，并在工具栏中添加easyui-searchbox控件。

1）表格easyui-datagrid

```
<table id="dg" class="easyui-datagrid" title="xxx列表" 
    style="width:99%;height:98%" 
    data-options="singleSelect:true,collapsible:true,url:'${ctx}/user/deadline/list',method:'get'"
    toolbar="#toolbar" 
    pagination="true" 
    rownumbers="true" 
    fitColumns="true" 
    singleSelect="true">
```

toolbar指向id为toolbar的html元素

2）自定义工具栏

```
<div id="toolbar">
    <a href="javascript:void(0)" class="easyui-linkbutton" 
        iconCls="icon-add" plain="true" onclick="newUser()">New User</a>
    <a href="javascript:void(0)" class="easyui-linkbutton" 
        iconCls="icon-edit" plain="true" onclick="editUser()">Edit User</a>
    <a href="javascript:void(0)" class="easyui-linkbutton" 
        iconCls="icon-remove" plain="true" onclick="destroyUser()">Remove</a>
    <!--查询工具-->
    <input id="ss" class="easyui-searchbox" style="width:300px" 
        data-options="searcher:search,prompt:'Please Input Value',menu:'#mm'"/>
    <!--查询工具菜单制定-->
    <div id="mm" style="width:120px">
        <div data-options="name:'username',iconCls:'icon-ok'">username</div>
    </div>
</div>
```

注：data-options中searcher指定了查询处理方法，menu指定了自定义菜单项

3）定义查询功能

```
//检索
function search(value,name){
   //alert(value+":"+name)
   if(value!=''){
        $('#dg').datagrid('load',{
            name:value
        });
   }
}
```

注：通过datagrid的load方法来查询数据。

4）问题

对于json对象来讲，冒号前面的视为属性，后面的视为属性值，属性值会被当成变量，因此能获取变量的值（即属性值就是变量值）。而前面的属性不会被当成变量来解析。

```
var searchString='{ "'+name+'":"'+value+'" }';
//var json=JSON.parse(searchString);
var json=$.parseJSON(searchString);
```

注：必须传入严格的json串，属性名称必须加双引号、字符串值也必须用双引号。

参考：http://www.jeasyui.net/tutorial/22.html