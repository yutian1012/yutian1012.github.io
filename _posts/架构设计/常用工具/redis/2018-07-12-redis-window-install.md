---
title: redis windows系统安装
tags: [redis]
---

参考：https://blog.csdn.net/u010137839/article/details/80210328

1）下载

目前官网只提供linux版本的下载

官网下载地址：http://redis.io/download

github下载地址：https://github.com/MSOpenTech/redis/tags

2）解压zip下载文件

解压后修改配置文件redis.windows.conf，搜索相关的选项，并在对应位置处修改

```
# 设置最大内存
maxmemory 1024000000
# 设置密码
requirepass 123456
```

注：可以看到默认端口port为6379

3）运行redis服务

cmd中运行下面的命令，或直接双击redis-server.exe

```
redis-server.exe redis.windows.conf
```

4）安装windows服务

cmd定位到redis的安装目录（解压目录）运行下面的指令

```
# 安装windows服务
redis-server --service-install redis.windows-service.conf --loglevel verbose
```

注：这里的配置文件是redis.windows-service.conf，上面的参数配置在这里也要修改一遍

打开服务（或运行services.msc），查看相关的redis服务的安装信息。

其他相关命令

```
# 卸载服务
redis-server --service-uninstall

# 开启服务
redis-server --service-start

# 停止服务
redis-server --service-stop
```

5）测试服务连接

使用安装目录下的redis-cli.exe客户端连接redis服务器，指定服务器地址和端口号即可连接

```
# 连接,-p和-h能够指定端口和ip，-a可以指定连接密码
redis-cli.exe -h 127.0.0.1 -p 6379

# 设置hello为key，world为值的缓存数据
set hello world
get hello
del hello

# 退出
exit
```

错误提示：

```
NOAUTH Authentication required.
```

解决方法

```
# auth命令，追加密码进行认证
auth 123456
```