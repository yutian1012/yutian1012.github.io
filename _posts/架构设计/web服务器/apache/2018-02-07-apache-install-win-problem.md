---
title: apache安装--windows常见问题
tags: [web服务器]
---

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