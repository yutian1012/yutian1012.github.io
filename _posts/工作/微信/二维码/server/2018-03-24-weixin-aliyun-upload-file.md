---
title: 阿里云服务上传文件
tags: [weixin]
---

参考：https://blog.csdn.net/qq_15807167/article/details/70904448

1）查看是否开启了ssh服务

```
netstat -tl
```

2）windows系统使用ssh方式上传文件

在windows系统上安装FileZilla，设置ssh连接，菜单：文件--站点管理--新建站点，协议选择SFTP，并设置登录的用户名和密码以及主机名。

![](/images/weixin/aliyun/ftp/sftp.png)