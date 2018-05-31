---
title: nginx安装
tags: [architecture]
---

参考：http://nginx.org/en/docs/windows.html

1）下载并解压

解压后进入解压目录D:\work\installer\nginx-1.14.0，运行ngnix.exe启动服务

```
cd D:\work\installer\nginx-1.14.0
# 运行命令
start nginx
# 查看nginx是否正常启动
tasklist /fi "imagename eq nginx.exe"
```

2）常用的操作命令

```
nginx -s stop   fast shutdown

nginx -s quit   graceful shutdown

nginx -s reload changing configuration,starting new worker processes with a new configuration,graceful shutdown of old worker processes

nginx -s reopen     re-opening log files
```

3）访问页面

```
http://localhost/
```

页面出现：Welcome to nginx!