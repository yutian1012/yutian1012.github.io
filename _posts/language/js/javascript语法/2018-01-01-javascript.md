---
title: js简介
tags: [js]
---

1）简介

javascript主要运用在浏览器端，处理用户的输入（输入数据的校验），并实现交互功能（通过DOM对象操作html页面元素）。javascript能够结合数据库组件、文件系统组件等扩展组件实现任何想要的功能。

javascript属于弱类型语言，数据类型在运行时决定，javascript大小写敏感，在定义变量时需要注意。

分号作为语句的结束符，可有可无，当一行只有一个程序语句时，结尾可以不使用分号。

常用编辑器：txt和aptana studio

2）浏览器对javascript的支持

为了能使javascript标准化，1997年发表了第一套脚本语言规范，即ECMA-262。新语言规范下javascript命名为ECMAScript。现在的浏览器都以ECMAScript为规范进行了支持，但不同浏览器的实现上存在部分差别。

检查浏览器版本

```
<script type="text/javascript">
    document.write("名称："+navigator.appName);
    document.write("版本号："+navigator.appVersion);
    document.write("发行代号："+navigator.appCodeName);
</script>
```