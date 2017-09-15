---
title: 流程表单数据
tags: [bpmx]
---

通过从ProcessCmd对象和ProcessRun对象中获取相关信息，封装成业务数据。处理过程查看ProcessRunService类的handlerFormData方法，该方法会向数据库中保持业务数据。

分支情况：根据processCmd中的getFormData是否为空，判断是否是批处理流程操作。

### 解析过程

ProcessRunService类的handlerFormData方法（处理在线表单数据）

```
/**
 * 处理在线表单数据。
 */
public BpmFormData handlerFormData(ProcessCmd processCmd, ProcessRun processRun, String nodeId) throws Exception {
    //页面填写的数据，封装成json传递到后台
    String json = processCmd.getFormData();
    //json提交表单数据空，一般对应的是批量处理的操作
    if (StringUtils.isEmpty(json)) {
        ...
    }

    //封装表单数据
    BpmFormData bpmFormData = null;
    //表单对应的表信息
    BpmFormTable bpmFormTable = null;
    //业务数据主键
    String businessKey = processCmd.getBusinessKey();
    // 判断节点Id是否为空，为空表示开始节点。
    boolean isStartFlow = StringUtil.isEmpty(nodeId) ? true : false;

    //下面的分支主要是实现了获取bpmFormTable对象
    //针对开始节点并且流程实例状态为草稿状态时
    if (isStartFlow && ProcessRun.STATUS_FORM.equals(processRun.getStatus())) {
        //通过表单信息获取bpmFormTable（流程自定义表信息bpm_form_table）
        //bpmFormTable包括主表和子表的信息
        Long formDefId = processRun.getFormDefId();
        BpmFormDef bpmFormDef = bpmFormDefService.getById(formDefId);
        //第二个参数表示是否获取隐藏的字段信息（bpm_form_field）
        bpmFormTable = bpmFormTableService.getByTableId(bpmFormDef.getTableId(), 1);
    } else {//针对其他节点获取bpmFormTable对象
        if (processRun.getParentId() > 0) {// 作为外部子流程
            //获取流程的变量
            Map<String, Object> variables = runtimeService.getVariables(processRun.getActInstId());
            //用于包含子流程的节点设置
            String parentActDefId = (String) variables.get(BpmConst.FLOW_PARENT_ACTDEFID);
            //获取对应子流程的表单定义信息
            bpmFormTable = bpmFormTableService.getByDefId(processRun.getDefId(), parentActDefId);
        } else {
            bpmFormTable = bpmFormTableService.getByDefId(processRun.getDefId(),"");
        }
    }

    // 有主键的情况,表示数据已经存在。
    if (StringUtil.isNotEmpty(businessKey)) {
        //获取表的主键字段名称
        String pkName = bpmFormTable.isExtTable() ? bpmFormTable.getPkField() : TableModel.PK_COLUMN_NAME;//判断是否是内部表
        //构造PkValue对象，存放主键信息（主键名称和主键值）
        PkValue pkValue = new PkValue(pkName, businessKey);
        //判断主键是否已经在使用了。
        //如果存在则证明业务表中已经有数据量，因此isADD字段为false，表示修改
        pkValue.setIsAdd(!bpmBusLinkService.checkBusExist(businessKey));
        //解析form提交表单信息
        bpmFormData = FormDataUtil.parseJson(json, pkValue, bpmFormTable);
    } else {
        bpmFormData = FormDataUtil.parseJson(json, bpmFormTable);
    }
    ...

    // 启动流程。
    if (isStartFlow) {
        // 保存表单数据,存取表单数据，并存储BpmBusLink业务数据关联信息
        bpmFormHandlerDao.handFormData(bpmFormData, processRun);
    } 
    ...
    return bpmFormData;
}
```

2）解析json数据

查看FormDataUtil类的parseJson方法。

```
public static  BpmFormData parseJson(String json,PkValue pkValue,BpmFormTable mainTableDef) throws Exception{
    JSONObject jsonObj= JSONObject.fromObject(json);
    BpmFormData bpmFormData=new BpmFormData(mainTableDef);
    if(pkValue==null)
        pkValue=generatePk(mainTableDef);
    bpmFormData.setPkValue(pkValue);
    //处理主表
    handleMain(jsonObj,bpmFormData);
    //子表
    handSub(jsonObj,bpmFormData);
    //意见
    handOpinion(bpmFormData, jsonObj);
    return bpmFormData;
}
```

### 对象内容

1）表信息

tableId（主表ID）
tableName（主表名称）
isExternal（是否外部表）
pkValue（主键对象）
dsAlias（数据源别名）

2）主表内容

mainFields（Map<String, Object>，主表字段和值）

3）子表信息

subTableList（List<SubTable>封装了子表对象的集合）

注：SubTable类包括子表名称，子表的主键名称，子表的外键名称，子表的数据集合dataList（List<Map<String, Object>>）

4）意见数据：

options（Map<String, String>）

5）流程变量数据：

variables（Map<String, Object>流程变量）

### 页面提交数据封装成json

查看taskStartFlowForm.jsp页面中定义的submitForm方法。

1）数据设置的核心代码

```
//获取自定义表单的数据
data=CustomForm.getData();
//设置表单数据，将数据信息设置到#formData控件上
$("#formData").val(data);
```

2）CustomForm.getData方法封装了json数据

CustomForm.js文件中的getData方法

```
// 主表数据，数据是使用字段名称:字段值的形式，封装到fields中的
var main = {
    fields:{}
};
//变量main表的控件，main.fields[name.replace(/.*:/, '')] = value;

//子表数据集合，可能存在多个子表
var sub = [];
//子表信息（for循环中）
var table = {
    tableName: $(this).attr('tableName'),
    fields: []
};
//变量子表的行数据（for循环中）
var row = {};
row[name] = value;
//将row信息放置到子表中
table.fields.push(row);
//将表中的数据放到子表集合中
sub.push(table);

//意见信息，所有name属性后置是opinion的控件都被视为意见信息
var opinion = [];
$('textarea[name^=opinion]').each(function() {
    opinion.push({
        name: $(this).attr('name').split(':')[1],
        value: $(this).val()
    });
});

//组合数据并返回
var data = {main: main, sub: sub, opinion: opinion};
return JSON2.stringify(data);
```

### 页面提交的隐藏域

查看taskStartFlowForm.jsp页面

```
<input type="hidden" include="1" name="curUserId" value="${curUserId}"/>
<input type="hidden" include="1" name="curUserName" value="${curUserName}"/>
<!-- 流程定义对象的主键 -->
<input type="hidden" include="1" name="actDefId" value="${bpmDefinition.actDefId}"/>
<input type="hidden" include="1" name="defId" value="${bpmDefinition.defId}"/>
<!-- 业务主键 -->
<input type="hidden" include="1" name="businessKey" value="${businessKey}"/>
<!-- 流程运行对象的主键 -->
<input type="hidden" include="1" name="runId" value="${runId}" />
<!-- 选择起始节点时设置使用该控件 -->
<input type="hidden" include="1" id="startNode" name="startNode" />
<!-- request的请求参数，?拼接传递过来的参数 -->
<c:if test="${not empty  paraMap}">
    <c:forEach items="${paraMap}" var="item">
        <input  include="1" type="hidden" name="${item.key}" value="${item.value}" />
    </c:forEach>
</c:if>
```

注：taskStartFlowForm.jsp是一个通用页面，如果要想正确的获取当前的是哪个流程，需要攒动actDefId或defId来标识流程信息。