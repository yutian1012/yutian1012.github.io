---
title: 单点登录源码查看
tags: [cas]
---

源码下载：https://github.com/apereo/cas

1）下载并编译

下载源码后解压文件，在根目录下执行mvn compile下载相应的依赖，并纠正一些错误的jar包，直到能够程序编译。然后将其导入到eclipse中查看。

注：在编译过程中可能有些jar包的repository地址发生了变化，需要在相关的pom.xml中配置相应的repository地址。

2）导入

导入方法：import--maven选择相应的项目地址。

![](/images/work/cas/cas_source.png)

注：导入的maven项目（Jasig-Central-Authentication-Service）是maven父模块，包含其他所有开发模块。

3）在eclipse中运行cas-server

将Jasig CAS Web Application部署到tomcat中，并运行。

![](/images/work/cas/deploy_cas_server.png)

访问：http://localhost:8080/cas/login

### cas登录功能分析

