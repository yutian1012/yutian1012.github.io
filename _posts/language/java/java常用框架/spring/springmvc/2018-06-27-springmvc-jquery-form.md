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