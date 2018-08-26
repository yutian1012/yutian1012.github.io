---
title: redis安装
tags: [redis]
---

1）安装过程

```
cd /usr/local/src
# 下载
wget http://download.redis.io/releases/redis-4.0.6.tar.gz
tar zxvf redis-4.0.6.tar.gz
cd redis-4.0.6

# 编译安装
make
make PREFIX=/usr/local/redis install
cd /usr/local/redis
# 复制配置文件到安装目录下
cp /usr/local/src/redis-4.0.6/redis.conf ./
# 启动
./bin/redis-server ./redis.conf

# 后台运行，修改配置文件
vim redis.conf
# 修改文件内容，以守护线程的方式启动
daemonize yes

# 查看进程
ps aux|grep redis

# 停止
./bin/redis-cli shutdown
```

2）在redis安装目录的bin下存在几个文件

redis-benchmark，redis的性能测试工具

redis-check-aof，检查aof日志的工具

redis-check-dump，检查rbd日志工具

redis-cli，连接用的客户端

redis-server，redis服务进程

3）客户端连接

```
# 启动客户端连接服务器
cd /usr/local/redis
./bin/redis-cli

# 通过ip和端口指定连接服务，-a可以指定连接密码
./bin/redis-cli -h xxx -p xxx

# 简单操作
# 设置变量site
set site www.zix.xdd
# 获取变量site的值
get site
```

4）错误信息

```
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
```

参考：https://blog.csdn.net/libraryhu/article/details/64920124

解决方法，在make命令中添加参数

```
make MALLOC=libc 
```

5）redis命令帮助文档

访问：https://redis.io/commands