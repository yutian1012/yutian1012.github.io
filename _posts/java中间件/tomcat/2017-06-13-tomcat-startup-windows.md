---
title: tomcat配置开机启动（windows）
tags: [tomcat]
---

参考：http://jingyan.baidu.com/article/a65957f4b12b8724e77f9b5a.html

问题引入

手动启动tomcat，运行tomcat安装目录/bin下的startup.bat，会出现一个类似dos的窗体，显示tomcat的启动运行内容。每次重新启动电脑后，都要手动执行startup.bat，非常不方便。

### 配置开机启动tomcat（windows）

1）打开tomcat的bin目录找到service.bat，在dos中运行命令

```
service.bat install tomcat 安装服务
#service.bat remove tomcat 删除服务
```

![](/images/middleware/tomcat/tomcat-install-service.png)

2）系统的服务中出现安装的tomcat服务（控制面板--管理工具--服务）

![](/images/middleware/tomcat/tomcat-windows-service.png)

3）在服务中设置开机启动(右击服务--属性)

![](/images/middleware/tomcat/tomcat-windows-startup.png)

4）查看进程

启动后可以dos中运行命令，查看其进程信息

```
tasklist | findstr tomcat
```

在任务管理器中也可以查看

![](/images/middleware/tomcat/tomcat-windows-manager.png)

5）命令启动

```
net start tomcat
```

注：tomcat的运行只能通过查看tomcat下的日志来分析。

### 问题：tomcat启动后服务不响应
