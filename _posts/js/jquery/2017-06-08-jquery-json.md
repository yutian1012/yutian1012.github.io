---
title: jquery-json字符串解析
tags: [jquery]
---

### $.parseJSON

参考：http://www.css88.com/jqapi-1.9/jQuery.parseJSON/

从jQuery3.0开始，不推荐使用$.parseJSON。要解析JSON字符串，请改用原生的JSON.parse方法。

注：JSON.parse兼容性，IE8以上均支持。

```
var searchString='{ "'+name+'":"'+value+'" }';
var json=JSON.parse(searchString);
```

解析时与$.parseJSON类似，要求字符串必须符合严格的JSON格式，即属性名称必须加双引号、字符串值也必须用双引号，如果值是整形可不加双引号。

### eval

参考：http://blog.csdn.net/yumay2009/article/details/51588160

```
var dataObj = eval("("+json+")");
```

注1：加上圆括号的目的是迫使eval函数在处理JavaScript代码的时候强制将括号内的表达式（expression）转化为对象，而不是作为语句（statement）来执行将json数据转换为json对象。

注2：使用eval函数解析JSON是一种很不安全的方式，能不用最好就不用，原因是eval不但可以解析JSON字符串，还会执行其中的代码块（如果有的话）