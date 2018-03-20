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

注：运行该工具，会在桌面的工具栏右下角出现监控图标，能够通过该监控快速启动，关闭apache服务。