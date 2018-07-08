---
title: jquery提交数组
tags: [jquery]
---

参考：https://blog.csdn.net/zhaohuijiadelu/article/details/54408324

1）方式一

使用JSON.stringify 将数组对象转化成json字符串;

```
var array = ["1", "2"];
$.ajax({  
    type: "POST",
    url: "/batches/migrate",
    dataType: "json",
    contentType: "application/json",
    data : JSON.stringify(array), 
    success : function(data) {  
        ...
    }  
});
```

注：除了jquery外，需要引用处理json数据的js文件

```
<script src="https://cdn.bootcss.com/json2/20160511/json2.min.js"></script>
```

后台接收数据

```
@RequestMapping(value="/migrate",method=RequestMethod.POST,consumes=MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Response migreate(@RequestBody Long[] batchId) {
    //batchService.migrate(batchIdList);
    return result(ExceptionMsg.SUCCESS);
}
```

错误使用情况

```
var array = ["1", "2"];
var param={"batchId":array};

$.ajax({  
    type: "POST",
    url: "/batches/migrate",
    dataType: "json",
    contentType: "application/json",
    data : JSON.stringify(param), 
    success : function(data) {  
        ...
    }  
});
```

注：这种方式传递的是json对象，并非json数组

2）方式二，服务器端处理

前端js代码

```
var array = ["1", "2"];
$.ajax({  
    type : 'POST',  
    url: path + '/check/testPost',
    contentType: "application/x-www-form-urlencoded",
    data: {"array": array},
    success : function(data) {  
    }  
}); 
```

后台代码

```
@RequestMapping(value = "/testPost", method = {RequestMethod.POST})
public void testPost(HttpServletRequest req) throws IOException {
    String[] array = req.getParameterValues("array[]");
    for (String string : array) {
        System.out.println(string);
    }
    return ;
}
```

注:两种post请求的content-type不同