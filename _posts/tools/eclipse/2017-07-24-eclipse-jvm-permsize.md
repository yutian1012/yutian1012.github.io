---
title: Myeclipse启动tomcat设置jvm内存
tags: [eclipse]
---

使用myeclipse部署web项目到tomcat后，在myeclipse中启动web项目。出现内存溢出错误

错误信息

```
严重: Error waiting for multi-thread deployment of directories to complete
java.util.concurrent.ExecutionException: java.lang.OutOfMemoryError: PermGen space
```

解决方案

菜单windows--preferences--MyEclipse--servers--tomcat--tomcat7.x--JDK，在Optional java VM arguments中设置内存大小。

```
-Xms1024m -Xmx1024m -XX:PermSize=256M -XX:MaxNewSize=256m -XX:MaxPermSize=512m -Djava.awt.headless=true
```

![](/images/tools/eclipse/tomcat-vm-optional.png)

注：具体值大小可根据系统配置进行参数设定。