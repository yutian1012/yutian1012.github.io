---
title: thymeleaf应用在script脚本中
tags: [spring]
---

1）在js中使用

参考：https://blog.csdn.net/mygzs/article/details/52667897

```
# 在script标签上使用th:inline="javascript"属性
# Thymeleaf解析时，会解析/*[[…]]*/的内容，并把获得的值替换掉/*[[…]]*/后面的内容
<script th:inline="javascript">
    /*<![CDATA[*/
      var valueType = /*[[${valueType}]]*/ 'defaultvalue';
    /*]]>*/
</script>
```