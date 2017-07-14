---
title: 流程表单中input值的value为空
tags: [bpmx]
---

问题：在执行流程时，经常发现流程表单的input内容为空（页面中value值为空，通过浏览器的查看页面元素的方式），再次保存时就会造成流程中的字段数据莫名被清空了。

原因是获取的数据全部以小写的形式放入到了map集合时，匹配时如果字段定义与map集合的key不同时，就无法找到value值。

1）流程节点页面

BpmFormHandlerService类的obtainHtml方法会调用获取权限信息

```
//加载数据，获取的数据全部以map集合来汇总的，
//并且数据都是以字段的小写做为map的key
BpmFormData data = getBpmFormData(bpmFormDef.getBpmFormTable(), pkValue, instanceId, actDefId, nodeId, isReCalcute,isPreView);
Map<String, Map> model = new HashMap<String, Map>();
//可以看到MainFields是Map类型的，并且key全部为表字段的小写
model.put("main", data.getMainFields());
model.put("opinion", data.getOptions());
model.put("sub", data.getSubTableMap());
```

2）设置值

流程页面的展示是通过freemarker来解析模板实现的。BpmFormHandlerService类的obtainHtml方法会调用freemarker引擎解析模板

```
// 传入控制器的service，用于在模版中解析字段。
map.put("service", bpmFormControlService);
...

String output = freemarkEngine.parseByStringTemplate(map, template);
```

注：解析模板时会使用到bpmFormControlService类，该类中提供了解析赋值input的方法。

3）为input控件赋值value

BpmFormControlService提供了getOptionValue（select控件赋值），getField（input控件赋值），getHiddenField（input type为hidden的控件赋值）。

以getHiddenField方法为例

```
Object object = model.get("main").get(fieldName);
String value = "";
if (object != null) {
    value = object.toString();
}
...

```

注：这里的fieldName如果不是全小写会无法匹配到map集合的key，导致获取值为null，最终输出到页面控件的value为空。

4）修改方法

```
Object object = model.get("main").get(fieldName);
//未获取值时，先转成小写查询
if(null==object){
    object=model.get("main").get(fieldName.toLowerCase());
}
//仍未获取值时，转成大写查询
if(null==object){
    object=model.get("main").get(fieldName.toUpperCase());
}
String value = "";
if (object != null) {
    value = object.toString();
}
```
