---
title: 流程表单中权限设置
tags: [bpmx]
---

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