---
title: webservice服务使用soapui工具
tags: [webservice]
---

### 1. 创建测试项目

在init wsdl中输入服务的wsdl地址

![](/images/java_structure/webservice/webservicesoapuiproject.png)

创建项目后会在左侧生成相应的服务接口

![](/images/java_structure/webservice/webservicesoapuiprojecttree.png)

可以继续在该项目中添加新的wsdl服务

右击项目--add wsdl

![](/images/java_structure/webservice/webservicesoapuiprojectaddwsdl.png)

![](/images/java_structure/webservice/webservicesoapuiprojectaddwsdltree.png)

### 2. 执行方法测试

双击项目中的服务方法，可以打开对应的发送请求框。

![](/images/java_structure/webservice/webservicesoapuirequest.png)

发生请求，修改request中的参数，然后发生

![](/images/java_structure/webservice/webservicesoapuiresponse.png)