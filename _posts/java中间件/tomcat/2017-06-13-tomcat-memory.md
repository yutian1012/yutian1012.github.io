---
title: tomcat设置jvm内存
tags: [tomcat]
---

在tomcat中运行一个大的web项目，有时经常会出现outofmemory错误。需要将jvm的内存设置的大一点。

参考：http://blog.csdn.net/alonesword/article/details/49361909

### tomcat设置内存

方式一：（适合手动运行startup.bat）

编辑catalina.bat文件，在文件开始位置处添加一行

```
set JAVA_OPTS= %JAVA_OPTS% -Xms2048m -Xmx2048m -XX:MaxPermSize=1024m
```

注：如果用startup.bat启动tomcat,OK设置生效

问题：

如果不是执行startup.bat启动tomcat而是利用windows的系统服务启动tomcat服务,上面的设置就不生效了（实际运行的是tomcat7.exe）。

方式二：（适合service.bat安装服务）

windows上的tomcat服务（bin\tomcat.exe）读取注册表中的值，而不是catalina.bat的设置。因此修改注册表。

注册表路径：HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Apache Software Foundation（64位机器的路径）

1）运行命令打开注册表

```
regedit
```

在注册表中查找java

![](/images/middleware/tomcat/regedit-find.png)

设置查找内容（查找字符串java）

![](/images/middleware/tomcat/regedit-find.png)

注：按F3查找下一个节点

![](/images/middleware/tomcat/tomcat-regedit-options.png)

2）修改内存参数

在Options项中编辑，并在最后添加

```
-Xms2048m
-Xmx2048m
-XX:MaxPermSize=1024m
```

注：需要一行设置一个参数

![](/images/middleware/tomcat/tomcat-regedit-options-edit.png)

### JVM中5个参数的理解

参考：http://unixboy.iteye.com/blog/174173/

1）通过new创建的对象的内存都在堆中分配，其大小可以通过-Xmx和-Xms来控制。

2）JVM用永久代（PermanetGeneration）来存放方法区，可通过-XX:PermSize和-XX:MaxPermSize来指定最小值和最大值。存放了要加载的类信息、静态变量、final类型的常量、属性和方法信息。（perm size 指的是永久代，也就是方法区）

```
 set JAVA_OPTS=-Xms64m -Xmx256m -XX:PermSize=128M -XX:MaxNewSize=256m -XX:MaxPermSize=256m
```

-Xmx 设置JVM最大可用内存
-Xms JVM初始分配的堆内存
-XX:PermSize= JVM初始分配的非堆内存（最小值）
-XX:MaxPermSize= JVM最大允许分配的非堆内存，按需分配
-XX:MaxNewSize= 当tomcat内存不足时，调用此分配

注：一般的要将-Xms和-Xmx选项设置为相同，而-Xmn为1/4的-Xmx值

注2：Perm为permanent永恒的，不变的。

![](/images/middleware/tomcat/tomcat-memory.png)
