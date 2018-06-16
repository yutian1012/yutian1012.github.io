---
title: 处理后台操作信息
tags: [project]
---

前台接收后台返回的操作响应信息，如操作成功/失败等信息。

1）方式一

后端处理响应的数据，然后更新前端数据，前端js通过判断相应状态，进而提示用户上一步的操作状态。

在HelloDemoController类中添加相关的处理方法

```
package com.favorites.favoriteswebdemo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloDemoController {
    /**
     * 跳转到order页面
     * @return
     */
    @RequestMapping("order")
    public String order() {
        return "order";
    }
    /**
     * 模拟提交订单页面
     * @param model
     * @return
     */
    @RequestMapping("submitOrder")
    public String submitOrder(Model model) {
        System.out.println("提交订单成功");
        model.addAttribute("oper", "SUCCESS");
        return "orderResult";
    }
}
```

创建order.html

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>订单提交测试页面</title>
</head>
<body>
    <a th:href="@{/submitOrder}">提交订单</a>
</body>
</html>
```

创建响应页面

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>

</body>
<script th:inline="javascript">
    /*<![CDATA[*/
        var oper=/*[[${oper}]]*/ 'FAIL';
        if(oper=='SUCCESS'){
            alert("提交订单成功！");
        }else{
            alert("提交订单失败!");
        }
        location.href="/order";
    /*]]>*/
</script>
</html>
```

注：这种方式的缺点的是需要后端处理返回数据，即需要模板引擎在返回前端时处理数据。使用jsp和freemark原理与之类似。

2）方式二

后台返回json数据，前台以ajax的形式提交，并在回调方法中判断操作结果。

修改order.html页面

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>订单提交测试页面</title>
</head>
<body>
    <a href="javascript:void(0);" 
        th:onclick="'javascript:submitOrder()'">ajax提交数据</a>
</body>
<script th:src="@{/webjars/jquery/2.0.0/jquery.min.js}"></script>
<script>
    function submitOrder(){
        $.post("/ajaxSubmitOrder",function(data){
            if(data.result=='SUCCESS'){
                alert("提交订单成功！");
            }else{
                alert("提交订单失败!");
            }
        });
    }
</script>
</html>
```

HelloDemoController类中添加ajaxSubmitOrder方法处理请求，并返回json数据

```
/**
 * ajax方式提交
 * @return
 */
@RequestMapping("ajaxSubmitOrder")
@ResponseBody
public Map<String,String> ajaxSubmitOrder(){
    System.out.println("提交订单成功");
    Map<String,String> result=new HashMap<>();
    result.put("result", "SUCCESS");
    return result;
}
```