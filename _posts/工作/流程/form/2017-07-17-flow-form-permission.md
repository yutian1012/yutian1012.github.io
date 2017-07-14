---
title: 流程表单中权限
tags: [bpmx]
---

1）流程节点页面

BpmFormHandlerService类的obtainHtml方法会调用获取权限信息

```
Map<String,Object> permission=bpmFormRightsService.getByFormKeyAndUserId(bpmFormDef, userId, actDefId, nodeId, parentActDefId,isReadOnly);
```

表单字段设置权限后会将记录存放到bpm_from_rights表中，获取表单权限方法

2）权限获取

BpmFormRightsService类的getByFormKeyAndUserId方法

```
public Map<String, Object> getByFormKeyAndUserId(BpmFormDef formDef,
        Long userId, String actDefId, String nodeId, String parentActDefId,boolean isReadOnly ) {
    
    String  formKey =formDef.getFormKey();

    //获取节点对应的权限，优先使用nodeId
    List<BpmFormRights> rightList = getFormRights(formKey, actDefId, nodeId, parentActDefId);
    
    if(BeanUtils.isEmpty(rightList) ){
        Map<String, Object>  permissionMap=getDefaultPermissionByFormKey( formDef, isReadOnly);
        return permissionMap;
    }
            
    Map<String, Object> permissions = new HashMap<String, Object>();
    
    Map<String, String> fieldPermission = new HashMap<String, String>();
    Map<String, String> opinionPermission = new HashMap<String, String>();
    Map<String, String> tablePermission = new HashMap<String, String>();
    Map<String, String> menuRightPermission = new HashMap<String, String>();
    List<JSONObject> subJsonList = new ArrayList<JSONObject>();
    List<JSONObject> subTableShowList = new ArrayList<JSONObject>();
    
    String right="w";
    
    CurrentUser currentUser=ServiceUtil.getCurrentUser();
    Map<String,List<Long>> relationMap=currentUserService.getUserRelation(currentUser,"formPermissionList");
    
    for (BpmFormRights rights : rightList) {
        JSONObject permission = JSONObject.fromObject(rights.getPermission());
        int type=rights.getType();
        String title=permission.getString("title");//字段
        
        if(BeanUtils.isNotIncZeroEmpty(userId)){
             right = getRight(permission,relationMap, userId);
        }
        if(isReadOnly && (right.equals("w") || right.equals("b"))){
             right="r";
        }
        
        if(right.equals("r") && permission.containsKey("rpost")){
            boolean rpost = permission.getBoolean("rpost");
            if(rpost)  right="rp";
        }
        
        if( permission.containsKey("menuRight")){
            JSONObject menuRight = permission.getJSONObject("menuRight");
            menuRightPermission.put(title, menuRight.toString());
        }
        switch (type) {
            //主表
            case BpmFormRights.FieldRights:
                fieldPermission.put(title.toLowerCase(), right);
                break;
            //子表字段
            case BpmFormRights.TableRights:
                JSONObject json=new JSONObject();
                json.put("tableName", permission.getString("tableName"));
                json.put("title", title);
                json.put("power", right);
                subJsonList.add(json);
                break;
            //意见字段
            case BpmFormRights.OpinionRights:
                opinionPermission.put(title, right);
                break;
            default:
                //子表权限 添加子表名称
                if(!permission.containsKey(title) && !permission.containsKey("tableName")){
                    permission.accumulate("tableName", title);
                }
                permission.remove("title");
                subTableShowList.add(permission);
                break;
        }
    }
    permissions.put("field", fieldPermission);
    permissions.put("opinion", opinionPermission);
    permissions.put("table", tablePermission);
    permissions.put("subFileJsonList", subJsonList);
    permissions.put("subTableShowList", subTableShowList);
    permissions.put("menuRight", menuRightPermission);
    
    return permissions;
}
```