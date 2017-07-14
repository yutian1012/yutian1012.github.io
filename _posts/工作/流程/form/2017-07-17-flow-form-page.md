---
title: 流程中执行的表单页面
tags: [bpmx]
---

### 表单页面的生成定位

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

### 表单页面的渲染

流程的表单页面主要分成2类，一类是草稿，另一类是流程各节点执行时生成的页面。

1）查看obtainHtml方法

BpmFormHandlerService类中的obtainHtml方法的实现。

```
@SuppressWarnings("rawtypes")
public String obtainHtml(BpmFormDef bpmFormDef, Long userId, String pkValue, 
    String instanceId, String actDefId, String nodeId, String ctxPath, String parentActDefId,boolean isReCalcute,boolean isPreView,boolean isReadOnly) throws Exception {
    
    //获取表单的设计模板
    String template = bpmFormDef.getTemplate();

    // 实例化BpmFormData数据
    BpmFormData data = getBpmFormData(bpmFormDef.getBpmFormTable(), pkValue, instanceId, actDefId, nodeId, isReCalcute,isPreView);
    
    Map<String, Map> model = new HashMap<String, Map>();
    model.put("main", data.getMainFields());
    model.put("opinion", data.getOptions());
    model.put("sub", data.getSubTableMap());

    Map<String, Object> map = new HashMap<String, Object>();
    map.put("model", model);
    // 传入控制器的service，用于在模版中解析字段。
    map.put("service", bpmFormControlService);
    //获取设置的权限信息
    Map<String,Object> permission=bpmFormRightsService.getByFormKeyAndUserId(bpmFormDef, userId, actDefId, nodeId, parentActDefId,isReadOnly);
    
    map.put("permission", permission);
    
    map.put("table", new HashMap<String, Object>());

    map.put("ctx", ctxPath); // 加入上下文路径

    String output = freemarkEngine.parseByStringTemplate(map, template);

    // 解析子表的默认内容
    //output = changeSubShow(output);

    String tabTitle = bpmFormDef.getTabTitle();
    // 有分页的情况。
    if (tabTitle.indexOf(BpmFormDef.PageSplitor) > -1) {
        output = getTabHtml(tabTitle, output);
    }
    // 替换流程实例分隔符。
    output = output.replace(INSTANCEID_SPLITOR, instanceId);
    // 流程图解析
    if (instanceId != "") {
        String repStr = "<iframe src=\"" + ctxPath + "/platform/bpm/processRun/processImage.ht?actInstId=" + instanceId + "&notShowTopBar=0\" name=\"flowchart\" id=\"flowchart\" marginwidth=\"0\" marginheight=\"0\" frameborder=\"0\" scrolling=\"no\" width=\"100%\"></iframe>";
        output = output.replaceAll(FLOW_CHART_SPLITOR, repStr).replaceAll(FLOW_CHART_SPLITOR_IE, repStr);
    } else if (actDefId != "") {
        String repStr = "<iframe src=\"" + ctxPath + "/platform/bpm/bpmDefinition/flowImg.ht?actDefId=" + actDefId + "\"  name=\"flowchart\" id=\"flowchart\" marginwidth=\"0\" marginheight=\"0\" frameborder=\"0\" scrolling=\"no\" width=\"100%\"></iframe>";
        output = output.replaceAll(FLOW_CHART_SPLITOR, repStr).replaceAll(FLOW_CHART_SPLITOR_IE, repStr);
    }

    // 子表权限放到页面 （在页面解析的）
    // 子表字段相关权限
    Map<String, Object> subFileJsonMap = new HashMap<String, Object>();
    
    subFileJsonMap.put("subFileJsonList", permission.get("subFileJsonList"));
    subFileJsonMap.put("subTableShowList", permission.get("subTableShowList"));
    
    
    String subFilePermissionMap = "{}";
    if (BeanUtils.isNotEmpty(subFileJsonMap)) {
        subFilePermissionMap = JSONObject.fromObject(subFileJsonMap).toString();
    }
    output += "<script type=\"text/javascript\" > var subFilePermissionMap = " + subFilePermissionMap + " </script>";
    return output;
}
```

2）查看表单设计

bpmFormDef.getTemplate()方法会获取自定义表单的定义信息，这是一个ftl模板。

流程管理--表单管理--自定义表单功能，设计出的表单页面会存放到bpmFormDef对象中。对应的表结构为bpm_form_def，该表中存在html和template两个字段，其中html是页面流程设计时使用的，而template是流程运行时自动生成表单使用的，两者基本上是相同的。

![](/images/work/bpmx/form/form_def_template.png)

3）表单展示数据获取

```
BpmFormData data = getBpmFormData(bpmFormDef.getBpmFormTable(), pkValue, instanceId, actDefId, nodeId, isReCalcute,isPreView);
```

BpmFormData对象中封装了对应表单的数据信息

```
bpmFormData.addSubTable(subTable);//子表数据
bpmFormData.setTableId(mainFormTableDef.getTableId());
bpmFormData.setTableName(tableName);//表明
bpmFormData.setPkValue(pk);//主表的主键信息
bpmFormData.addMainFields(mainData);//主表数据
bpmFormData.setOptions(formOptions);//表单意见内容
```

注：主表数据和子表数据是在流程有了业务主键后才会去查询的（不是起始节点）。而如果是起始节点，则会根据自定义表时设置的流水号和脚本计算出结果设置到主表中。

4）执行freemarker解析

```
//用于封装freemarker引擎执行时的变量
Map<String, Object> map = new HashMap<String, Object>();
//model为上一步获取的表单数据信息
map.put("model", model);
// 传入控制器的service，用于在模版中解析字段。解析模板时使用的对象
map.put("service", bpmFormControlService);
//获取设置的权限信息
Map<String,Object> permission=bpmFormRightsService.getByFormKeyAndUserId(bpmFormDef, userId, actDefId, nodeId, parentActDefId,isReadOnly);
map.put("permission", permission);

map.put("table", new HashMap<String, Object>());
map.put("ctx", ctxPath); // 加入上下文路径

//解析模板获取输出html
String output = freemarkEngine.parseByStringTemplate(map, template);
```

注：FreemarkEngine对象在使用时对原生的freemarker解析进行了封装，具体封装内容可以查看该类的实现（com.hotent.core.engine.FreemarkEngine）。

5）具体的解析过程

通过格式化数据库表bpm_form_def的template字段可以看到，每一个html中的input字段都会被freemarker语句包含着。

```
<#assign approvalresultHtml><input name="m:t_idea:approvalresult" id="result" type="hidden" value="#value" /> </#assign> 
${service.getHiddenField('approvalresult',approvalresultHtml, model, permission)}
```

注：在freemarker解析时使用的service是在map中设置的bean实例，可以看到${}插值操作会调用service的getHiddenField方法。

查看BpmFormControlService类的getHiddenField方法

```
public String getHiddenField(String fieldName, String html,
        Map<String, Map<String, Map>> model,
        Map<String, Map<String, String>> permission) {

    //fieldName = fieldName.toLowerCase();
    // 取得值，从主表中获取fieldName的值
    Object object = model.get("main").get(fieldName);
    String value = "";
    if (object != null) {
        value = object.toString();
    }
    //根据用户设置的权限获取该字段的权限
    String right = this.getFieldRight(fieldName, permission);
    if ("w".equals(right)) {//写权限
        return StringUtil.replaceAll(html, "#value", value);
    } else if ("b".equals(right)){//必填
        html=toRequired(html);
        return StringUtil.replaceAll(html, "#value", value);
    }else if ("r".equals(right)) {// 只读和其他情况下不返回值
        String rpostInput = getReadPost(html);
        rpostInput=StringUtil.replaceAll(rpostInput, "#value", value);
        return rpostInput;
    }else {
        return NO_PERMISSION;
    }
}
```

注：权限字符串，w写权限，b必填，r只读，rp只读提交。

注2：通过运行该方法后会生成相应的html字符串，并最终输出到页面上。