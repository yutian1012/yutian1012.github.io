---
title: 流程表单中权限设置
tags: [bpmx]
---

### 权限的使用

1）流程启动和流程办理页面

流程启动页面taskStartFlowForm.jsp，表单的展示通过${form}，从后台直接获取html页面进行的展示操作

```
<div class="panel-detail" type="custform" id="custformDiv">${form}</div>
```

流程办理页面taskToStart.jsp，与启动流程页面的实现方式是类似的

```
<div id="custformDiv" class="printForm" type="custform">
    ${form}
</div>
```

2）处理过程controller，流程的控制层类主要涉及到taskController。

启动流程页面的处理方法为taskController类的StartFlowForm方法。流程启动页面的来源主要有两部分（草稿，和新建）。

来源一：草稿页面

```
form = bpmFormHandlerService.obtainHtml(bpmFormDef, sysUser.getUserId(), businessKey, "", actDefId, nodeId, ctxPath, "", true,false,false);
```

来源二：新建页面

```
FormModel formModel = getStartForm(bpmNodeSet, businessKey, actDefId, ctxPath,isReCalcuate);
...
form = formModel.getFormHtml();//获取表单的html，form为string类型的值
```

来源三：办理流程页面

处理方法为taskController类的toStart方法，会调用getToStartView方法。

```
FormModel formModel = bpmFormDefService.getNodeForm(processRun, nodeId, tempLaunchId, ctxPath, variables,false);
...
form = formModel.getFormHtml();
```

注：这里的FormModel获取方式是通过getNodeForm，即获取节点的form表单。

### 补充，form表单类型

表单类型主要分为两种：在线的表单和外部表单。查看BpmConst类定义的常量值

```
/**
 * 在线的表单
 */
public static final Short OnLineForm=0;
/**
 * 外部表单
 */
public static final Short UrlForm=1;
```

### 表单权限的设置（自定义表单设置）

在自定义表单的列表功能处可以设置表单流程的表单权限。表单的权限包括：隐藏，只读，编辑和必填。对应的值为：1，2，3，4，即隐藏权限对应值为1。

1）编辑权限页面

bpmFormRightsDialog.jsp页面，controller处理类BpmFormRightsController类的dialog方法。

页面bpmFormRightsDialog.jsp中的js方法加载表单字段即权限设置。

```
//权限处理
__Permission__=new Permission();
//设置节点权限。
if(isNodeRights){
    __Permission__.loadByNode(actDefId,nodeId,formKey,parentActDefId);
    $("#nodeId").change(changeNode);
}else if(actDefId){
    __Permission__.loadByActDefId(actDefId,formKey,parentActDefId);
}else{
    //根据表单key获取表单权限。
    __Permission__.loadPermission(formKey);
}
```

查看Permission.js文件中的loadPermission方法，会调用BpmFormRightsController类的getPermissionSetting方法。会到bpm_form_rights表中查询设置记录信息。

```
public Map<String, List<JSONObject>> getPermission(String formKey, String actDefId, String nodeId, String parentActDefId) {
    Map<String, List<JSONObject>> map = new HashMap<String, List<JSONObject>>();
    List<BpmFormRights> rightList =getFormRights(formKey, actDefId, nodeId, parentActDefId);
    BpmFormDef bpmFormDef = bpmFormDefService.getDefaultVersionByFormKey(formKey);
    if (BeanUtils.isEmpty(rightList)) {         
        map = getPermissionJson(bpmFormDef);
    }else{
        map = getPermissionJson(rightList,bpmFormDef);
    }
    return map;
}
```

注：查询时如果不传递nodeId，actDefId等信息，则查询时只会根据formKey字段查询。

```
SELECT id,formKey,name,permission,type,actDefId,parentActDefId,nodeId FROM bpm_form_rights WHERE formKey='ipph_cygl' and (actDefId is null or actDefId='') and (NODEID IS NULL OR NODEID ='') ;
```

注：通过查询语句，可以看到，该查询会过滤掉在节点和流程上设置的权限。

注2：对自定义表单的权限设置可理解为一个全局的权限设置