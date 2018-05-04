---
title: spring boot 使用extjs前端框架
tags: [spring]
---

Ext JS是一个流行的JavaScript框架，它为使用跨浏览器功能构建Web应用程序提供了丰富的UI。常用于企业内部管理系统、行政系统等SAP类型的应用。

1）spring boot静态资源

a.Spring Boot对静态资源映射提供了默认配置。

```
# Spring Boot默认将/**所有访问映射到以下目录
classpath:/static
classpath:/public
classpath:/resources
classpath:/META-INF/resources
```

b.创建目录：

在resources目录下新建public目录，用来存放静态资源数据。

注：SpringBoot默认会挨个从public，resources，static里面找是否存在相应的资源，如果有则直接返回。

2）引入extjs

下载extjs压缩包，解压，需要使用的文件都位于build目录下。



