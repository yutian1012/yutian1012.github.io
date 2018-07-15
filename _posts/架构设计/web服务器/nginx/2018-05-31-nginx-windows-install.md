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
# fast shutdown
nginx -s stop   

# graceful shutdown
nginx -s quit   

#changing configuration,starting new worker processes with a new configuration,graceful shutdown of old worker processes

nginx -s reload 

# re-opening log files
nginx -s reopen   

# 帮助信息查看
nginx -?  
```

注：-s表示signal，信号量的意思

3）访问页面

```
http://localhost/
```

页面出现：Welcome to nginx!

4）日志

在ngnix解压目录下存在logs目录，该目录下有两个文件access.log和error.log文件，如果访问出现错误时，可以查看error.log文件。