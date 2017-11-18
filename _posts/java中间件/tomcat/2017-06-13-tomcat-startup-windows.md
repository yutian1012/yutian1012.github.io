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

### 问题：tomcat启动出现内存溢出

1）解决方式一：修改注册表

参考：http://blog.csdn.net/lfsf802/article/details/46700553

打开注册表HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Apache Software Foundation\Procrun 2.0\Tomcat6\Parameters\Java(路径可能有一点点差别)中的Options。

![](/images/middleware/tomcat/tomcat-rededit-option.png)

2）解决方式二：通过tomcat提供的tomcat7w.exe程序设置

程序：%TOMCAT_HOME%/bin/tomcat7w.exe

![](/images/middleware/tomcat/tomcat-tomcatw-exe.png)

注：Initial memory pool就是初始化堆内存大小，Maximun memory pool是最大堆内存大小。

而要设置Perm Gen池的大小就要在Java Option里面加参数了，在里面加上：

```
-Dcatalina.base=%tomcat_home%
-Dcatalina.home=%tomcat_home%
-Djava.endorsed.dirs=%tomcat_home%\endorsed
-Djava.io.tmpdir=%tomcat_home%\temp
-XX:PermSize=128M
-XX:MaxPermSize=512M
```

3）解决方式三：使用jdk1.8

参考：https://yq.aliyun.com/articles/48957

JDK8 HotSpot JVM 使用本地内存来存储类元数据信息并称之为：元空间（Metaspace）；意味着不会再有java.lang.OutOfMemoryError:PermGen问题，也不再需要你进行调优及监控内存空间的使用。

Metaspace容量：

默认情况下，类元数据只受可用的本地内存限制（容量取决于是32位或是64位操作系统的可用虚拟内存大小）。

新参数（MaxMetaspaceSize）用于限制本地内存分配给类元数据的大小。如果没有指定这个参数，元空间会在运行时根据需要动态调整