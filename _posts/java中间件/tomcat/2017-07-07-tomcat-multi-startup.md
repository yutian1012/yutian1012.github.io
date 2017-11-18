---
title: 一个系统中启动多个tomcat
tags: [tomcat]
---

实例：在windows系统分别启动tomcatA和tomcatB

1）准备tomcat

解压2次tomcat的安装包（zip文件），将解压文件夹分别重命名为tomcatA和tomcatB。

2）修改启动文件中的环境变量CATALINA_HOME

设置不同的tomcat下启动的CATALINA_HOME的安装目录不同，如tomcatA下的CATALINA_HOME的值为tomcatA的安装目录，tomcatB下的CATALINA_HOME的值为tomcatB的安装目录。

在startup.bat文件中设置

```
set CATALINA_HOME=E:/apache-tomcat-7.0.82
```

注：确定出启动的服务路径

3）修改tomcat的server.xml端口（避免端口冲突）

```
<Server port="8005" shutdown="SHUTDOWN">
...
<Connector port="8080" protocol="HTTP/1.1" redirectPort="8443"
...
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

注：redirectPort的端口可以不修改。其他3个端口需要修改。

4）重名名打开的tomcat窗口

参考：http://blog.csdn.net/lv_shijun/article/details/52541000

编辑catalina.bat文件，查询title位置，并修改其值。

```
if "%TITLE%" == "" set TITLE=Tomcat
```