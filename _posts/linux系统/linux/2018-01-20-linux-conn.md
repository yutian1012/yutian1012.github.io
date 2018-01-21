---
title: linux客户端连接
tags: [linux]
---

在连接服务器是一定要使用ssh进行连接，因此服务器必须安装ssh服务，并监听相应的端口（默认23）。

1）设置服务器网络

```
# 启动网卡
ifup eth0
# 查看是否能够正常连接
ping baidu.com
```

2）linux服务器安装ssh服务

最常用的提供Linux远程连接服务的工具就是SSH软件了，SSH分为SSH客户端和SSH服务器端两部分。其中，SSH服务器端包含的软件程序主要有openssh和openssl。

首先判断是否安装了SSH服务器端工具

```
# 判断是否安装ssh服务
rpm -qa openssh openssl
# yum方式安装
yum install openssh-server
```

注：openssh是提供SSH服务的程序，openssl是为SSH提供连接加密的程序。

2）使用客户端连接软件连接

