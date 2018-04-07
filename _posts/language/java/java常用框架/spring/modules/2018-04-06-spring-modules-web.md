---
title: spring框架的web模块
tags: [spring]
---

spring-web模块中，包括的jar文件有：web，web-servlet，web-portlet，websocket等jar包。

1）mvc的支持

MVC（Model-View-Controller）模式是一种普遍被接受的构建Web应用的方法，它可以帮助用户将界面逻辑与应用逻辑分离。Java从来不缺少MVC框架，Apache的Struts、JSF、WebWork和Tapestry都是可选的最流行的MVC框架。虽然Spring能够与多种流行的MVC框架进行集成，但它的Web和远程调用模块自带了一个强大的MVC框架，有助于在Web层提升应用的松耦合水平。

2）远程调用方案

除了面向用户的Web应用，该模块还提供了多种构建与其他应用交互的远程调用方案。Spring远程调用功能集成了RMI（RemoteMethodInvocation）、Hessian、Burlap、JAX-WS，同时Spring还自带了一个远程调用框架：HTTPinvoker。Spring还提供了暴露和使用RESTAPI的良好支持。

### jar包信息

1）spring-web.jar

SpringWeb下的工具包，包括自动载入Web Application Context 特性的类、Struts 与JSF 集成类、文件上传的支持类、Filter 类和大量工具辅助类。

2）spring-web-portlet.jar

基于protlet的MVC实现

3）spring-web-servlet.jar

基于servlet的MVC实现

4）spring-web-struts.jar

整合Struts的时候的支持

5）spring-websocket.jar

Spring提供了对Web Socket编程的支持，包括支持JSR-356——Java API for WebSocket；

鉴于Web Socket仅仅提供了一种低层次的API，急需高层次的抽象，因此Spring4.0在Web Socket之上提供了一个高层次的面向消息的编程模型，该模型基于SockJS，并且包含了对STOMP协议的支持；