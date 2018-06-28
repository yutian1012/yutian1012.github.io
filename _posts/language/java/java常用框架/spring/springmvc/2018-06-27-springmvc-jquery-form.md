---
title: spring mvc处理jquery提供的集合类型的表单
tags: [spring]
---

参考：https://blog.csdn.net/freeniuniu/article/details/78806508

1）错误信息

```
Property referenced in indexed property path 'fieldList[0][forLog]' is neither an array nor a List nor a Map; 
```


2）原因

JQuery 会将你的参数映射成这样：

```
beans[0][agreementId]=1
beans[0][answerId]=2
```

问题是Spring MVC需要这种的参数格式：

```
beans[0].agreementId=1
beans[0].answerId=2
```

3）使用插件方式提交

```
$.ajax({
  url:"ajax/mycontroller.do", 
  type: "POST", 
  data: JSON.stringify( objecdt ), 
  success: callback, 
  dataType: "json",
  contentType: "application/json"
}); 
```

注：利用JSON插件完成序列化操作。

引入相关的json脚本

```
<script src="https://cdn.bootcss.com/json2/20160511/json2.js"></script>
```

4）spring mvc后台接收json

```
@RequestMapping(value="/saveTable",method=RequestMethod.POST,
    consumes=MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Response saveTable(@RequestBody TableModel table) {
    tableService.save(table);
    //return "redirect:/tables/add";
    return result(ExceptionMsg.SUCCESS);
}
```

注：后端接收json，使用@RequestBody注解实现对象的封装操作