---
title: swagger
tags: [spring]
---

参考：https://www.cnblogs.com/mao2080/p/8991027.html

1）背景

随着互联网技术的发展，现在的网站架构基本都由原来的后端渲染，变成了：前端渲染、先后端分离的形态，而且前端技术和后端技术在各自的道路上越走越远。 前端和后端的唯一联系，变成了API接口；API文档变成了前后端开发人员联系的纽带，变得越来越重要，swagger就是一款让你更好的书写API文档的框架，而且swagger可以完全模拟http请求，入参出参和实际情况差别几乎为零。

2）什么是swagger

swagger是编写服务端api文档的框架集，方便使用者根据api文档调用服务的手册。能够提供api文档的展示，并能通过swagger ui实现浏览器访问和调试接口的作用。

swagger最明显的就是代码移入性比较强，需要在服务器端的实体类（与前端交换，用于接收前端传递的参数）和相应的controller方法上添加相应的注解。

3）官网

地址：https://swagger.io/

API官方文档：http://springfox.github.io/springfox/docs/current/

4）swagger的生态圈

参考：https://blog.csdn.net/i6448038/article/details/77622977