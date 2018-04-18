---
title: 持续集成jenkins安装
tags: [jenkins]
---

Jenkins是一个独立的开源自动化服务器，可用于自动化各种任务，如构建，测试和部署软件。Jenkins可以通过本机系统包Docker安装，甚至可以通过安装JavaRuntimeEnvironment的任何机器独立运行。

1）下载软件

下载地址：https://jenkins.io/download/

这里使用Generic Java package (.war)作为运行工具

注：jenkins.war工具只是完成了web应用，能够提供web页面的访问，而具体的集成工作需要多个插件配合使用（如ant，git，docker等）。

2）运行jenkins

前提条件机器安装了jvm（java运行时环境）。

```
java -jar jenkins.war --httpPort=8080
```

访问服务：http://localhost:8080

3）安装过程

启动后，会要求输入密码，密码文件位于${home}/.jenkins\secrets\initialAdminPassword文件中。

输入密码后，会要求选择安装jenkins的插件，因为jenkins的运行时需要多个插件配合使用来完成具体工作。安装的插件会放置在${home}/.jenkins\plugins目录下。

![](/images/tools/jenkins/jenkins-plugin.png)

注：配置过程中的数据信息都会保存到${home}/.jenkins目录下，该目录相当于工作目录。

![](/images/tools/jenkins/console.png)