---
title: 流程表单中数值类型用逗号分隔成各式化
tags: [bpmx]
---

1）场景

在表单中设置的一些数值型的字段，如10000014530031显示到页面上样式为10,000,014,530,031，即显示的时候对数值进行了处理，类似金额的3个为一组的分隔处理。有时候这种方式的处理，并不是我们希望的。

2）解决方法：

在自定义表页面中，设置字段类型为number，但不要勾选千分位。

![](/images/work/bpmx/form/form-field-number.png)

该字段的设置会存放到bpm_form_field表的ctrlproperty字段中，json形式存放，json的key为isShowComdify。

3）源码查看

加载页面BpmFormHandlerService类的obtainHtml方法，会调用相应的数据存储信息的获取方法。使用BpmFormHandlerDao类的getByKey方法获取数据。获取数据后，会根据字段BpmFormField字段的定义信息来格式化显示查询到的数值。

查看BpmFormHandlerDao类的handMap方法

```
// 对时间字段单独处理。
if (obj instanceof Date) {
    String format = "yyyy-MM-dd";
    String confFormat = bpmFormField.getPropertyMap().get("format");
    if (StringUtil.isNotEmpty(confFormat)) {
        format=confFormat;
    }
    String str = TimeUtil.getDateTimeString((Date) obj, format);
    rtnMap.put(key, str);
} 
else if(obj instanceof Number){
    Map<String,String> paraMap=bpmFormField.getPropertyMap();
    Object isShowComdify= paraMap.get("isShowComdify");
    Object decimalValue= paraMap.get("decimalValue");
    Object coinValue= paraMap.get("coinValue");
    
    String str =StringUtil. getNumber(obj,isShowComdify,decimalValue,coinValue);
    rtnMap.put(key, str);
}
else {
    
    //附件替换
    if( BeanUtils.isNotEmpty(bpmFormField.getControlType()) && bpmFormField.getControlType().shortValue() ==FieldPool.ATTACHEMENT ){
        obj = ((String) obj).replace("\"", "￥@@￥");
    }   
    rtnMap.put(key, obj);
}
```

注：该方法会根据不同的字段类型设置，对存储数值进行格式化显示。