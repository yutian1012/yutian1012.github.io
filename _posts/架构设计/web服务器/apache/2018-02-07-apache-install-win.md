---
title: apache安装--windows
tags: [web服务器]
---

### windows系统安装apache（2.4）

1）下载apache

下载地址：http://httpd.apache.org/download.cgi

注：下载的都是免安装版本的zip压缩包。

2）解压文档并查看目录结构

```
# 存放常用命令，如httpd
─bin

#存放配置文件，如httpd.conf
─conf
 
# 存放Apache启动或关闭时的错误记录
─error

# 存放站点的文件夹，该文件夹初始只有一个index.html文件
─htdocs

# 存放站点图标
─icons

# 头文件
─include

# lib库
─lib

# 记录日志
─logs

# Apache的核心，存放Apache模块。Apache是以模块的方式来管理功能。
─modules
```

3）运行

```
# 切换到apache安装目录的bin下
cd Apache_Home/bin
# 首先要先安装apache服务
httpd -k install
# 启动服务可以通过services.msc（服务窗口）或ApacheMonitor.exe程序
```

4）监控服务的运行

```
# 运行apache/bin下的ApacheMonitor.exe程序
```

### 安装问题：

1）无法启动此程序，因为计算机中丢失VCRUNTIME140.dll。尝试重新安装该程序以解决此问题。

![](/images/web/apache/vcruntime.png)

解决方法，下载vc相应的运行库，并安装。下载文件vc_redist.x64.exe。在https://www.apachehaus.com/cgi-bin/download.plx页面的最下端给出了下载地址链接。

![](/images/web/apache/vcruntime-lib.png)

2）无法安装apache服务

在cmd中执行httpd -k install命令，提示无法安装。

解决方法：以管理员的身份运行cmd。

3） ServerRoot must be a valid directory

修改httpd.conf配置文件中的ServerRoot。将ServerRoot应该指向Apache的根目录，即安装目录。

4）问题：'xxx/Apache24/htdocs' is not a directory, or is not readable

修改SRVROOT指向Apache的根目录。

5）问题：no listening sockets available, shutting down

主要是因为apache需要的端口被其他程序占用，这里，apache需要443端口，而该端口被vmware-host.exe程序占用。