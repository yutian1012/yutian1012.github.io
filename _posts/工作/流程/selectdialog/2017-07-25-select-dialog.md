---
title: 自定义查询对话框--java脚本
tags: [bpmx]
---

自定义对话框的预览功能，页面bpmFormDialogShowFrame.jsp，提交查询功能查看BpmFormDialogController类的showFrame方法。

1）获取查询对话框的设置

```
@RequestMapping("showFrame")
public ModelAndView showFrame(HttpServletRequest request, HttpServletResponse response) throws Exception {
    String alias = RequestUtil.getString(request, "dialog_alias_");
    BpmFormDialog bpmFormDialog =bpmFormDialogService.getByAlias(alias);
    ...
    bpmFormDialog = bpmFormDialogService.getData(bpmFormDialog, paramsMap);
    ...
}
```

注：通过对话框的别名字段获取设置的对话框信息，然后根据对话框设置信息以及查询条件获取数据。

2）获取查询数据

查看bpmFormDialogService类的getData方法。该方法内部封装了数据的获取过程，将获取的数据信息封装到bpmFormDialog对象中。

```
public BpmFormDialog getData(BpmFormDialog bpmFormDialog, Map<String, Object> params) throws Exception {
    ...
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("objectName", bpmFormDialog.getObjname());//表名
    map.put("displayList", bpmFormDialog.getDisplayList());//显示字段集合
    map.put("conditionList", bpmFormDialog.getConditionList());//条件字段集合
    map.put("sortList", bpmFormDialog.getSortList());//排序字段集合
    ...
    String sql = ServiceUtil.getSql(map, params);
    List list = JdbcTemplateUtil.getPage(bpmFormDialog.getDsalias(), currentPage, pageSize, sql, params);
    ...
    bpmFormDialog.setPageBean(((PageList)list).getPageBean() );
}
```

![](/images/work/bpmx/select-dialog/dialogfieldset.png)

注：map中封装了查询对话框的字段设置信息。调用ServiceUtil.getSql获取sql语句，使用jdbcTemplate执行sql，获取数据。

3）拼接sql语句

```
public static String getSql(Map<String, Object> map, Map<String, Object> params) {
    //数据准备
    String objectName=(String) map.get("objectName");
    List<DialogField> retrunList=(List<DialogField>) map.get("returnList");
    List<DialogField> displayList=(List<DialogField>) map.get("displayList");
    List<DialogField> conditionList=(List<DialogField>) map.get("conditionList");
    List<DialogField> sortList=(List<DialogField>) map.get("sortList");

    //拼接sql
    String sql = "select ";
    if (BeanUtils.isEmpty(retrunList)) {
        sql = "select a.* from " + objectName + " a";
    } else {
        String returnStr = "";
        for (DialogField dialogField : retrunList) {
            returnStr += "," + dialogField.getFieldName();
        }
        returnStr = returnStr.replaceFirst(",", "");
        sql += returnStr + " from " + objectName;
    }
    //获取条件
    String where = getWhere(conditionList, params);
    
    String orderBy = " order by ";
    ...
}
```

注：执行完成后会获取待执行的sql信息。getWhere方法支持字段设置和java脚本方式。

4）java脚本获取where条件

核心思想是通过groovy脚本引擎执行定义好的java脚本，并将页面参数传递到脚本引擎变量中

```
//定义的脚本信息
String java = dialogField.getDefaultValue();
if (StringUtil.isNotEmpty(java)) {
    GroovyScriptEngine groovyScriptEngine = (GroovyScriptEngine) AppUtil.getBean(GroovyScriptEngine.class);
    Map<String,Object> paramsMap = new HashMap<String,Object>() ;
    //参数信息是以map为key的集合对象
    paramsMap.put("map", params) ;
    value = groovyScriptEngine.executeObject(java, paramsMap);
}
```

注：GroovyScriptEngine类封装了脚本的执行。

脚本引擎中参数的获取

```
if(map.get("ORGNAME")!=null){
    sql+=" and ORGNAME LIKE '%"+map.get("ORGNAME")+"%'";
}
```

注：脚本应拼接并返回任意的可执行的sql语句的where条件。