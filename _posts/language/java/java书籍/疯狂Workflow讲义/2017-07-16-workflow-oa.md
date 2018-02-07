---
title: 疯狂workflow讲义OA源码环境搭建和运行
tags: [activiti]
---

1）源码下载

源码下载地址：http://www.broadview.com.cn

2）安装数据库

书中的OA项目数据库使用的是mysql。

3）安装tomcat

压缩包版tomcat安装后设置环境变量CATALINA_HOME

4）将源码导入到eclipse中

错误信息提示：

```
Unbound classpath variable: 'TOMCAT_HOME/lib/annotations-api.jar' in project 'oa'
```

![](/images/book/workflow/oa/oa_buildpath_error.png)

解决方法：

在eclipse的buildpath中设置变量

![](/images/book/workflow/oa/set_buildpath_variable)

5）配置数据库连接

数据库连接的设置位于WEB-INF目录下的applicationContext.xml文件中。

6）运行系统

用户信息查看数据库表act_id_user，包含用户名和密码。