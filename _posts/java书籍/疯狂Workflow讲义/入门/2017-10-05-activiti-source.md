---
title: activiti源码下载查看
tags: [activiti]
---

参考：http://blog.csdn.net/zcy860511/article/details/39529417

1）下载源码

Activiti源码托管在Github上，需要通过Git进行代码的下载。

下载地址：https://github.com/Activiti/Activiti

2）导入myeclipse中查看

项目是使用maven组织的，因此在导入前可先将项目进行构建，然后再导入。

进入解压目录，执行命令

```
mvn compile
```

3）Eclipse提示@Override需要删除的问题

windows->preferences->java->Complier->Compiler compliance settings选择1.6即可。

注：下载源码时最好下载最新源码，否则maven构建项目时经常出现jar包无法获取，即一些依赖的jar包在不同仓库的位置发生了改变，导致不能正确的编译出来。如果使用新版本，这个问题就不会发生。