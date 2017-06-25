---
title: 页面提交typeMismatch
tags: [work]
---

### 原生类型typeMismatch错误

服务器端存在一个pojo对象，对象中包含了一个float的属性变量（这里使用的是float原生类而不是Float包装类型）。页面提交数据时抛出异常。

错误信息

```
Field error in object 'TFeeItem' on field 'personalReduceRate': rejected value []; 
codes [typeMismatch.TFeeItem.personalReduceRate,typeMismatch.personalReduceRate,typeMismatch.float,typeMismatch];
 arguments [org.springframework.context.support.DefaultMessageSourceResolvable: 
 codes [TFeeItem.personalReduceRate,personalReduceRate]; arguments []; 
 default message [personalReduceRate]]; 
 default message [Failed to convert property value of type 'java.lang.String' to required type 'float' for property 'personalReduceRate'; nested exception is java.lang.NumberFormatException: empty String]] with root cause
```

注：主要意思是空字符串在转换到float对象属性时发现类型不匹配。

解决方法

将属性的类型float改成包装类型Float即可。