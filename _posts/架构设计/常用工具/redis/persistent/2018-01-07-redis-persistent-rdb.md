---
title: redis的rdb持久化方式
tags: [redis]
---

rdb持久化方式又称为快照持久化方式。

快照就是将内存的数据原样不动的直接写入磁盘中，存入的是内存映像文件。每隔N分钟或N次写操作后，从内存dump数据形成rdb文件，压缩并放在备份目录。

注：其中N分钟，N次写操作，压缩，备份目录等参数都是可以配置的选项。

1）配置文件redis.conf

```
# dump的频率，是由时间和更改频率相互作用的
# 900秒（15分钟）之后至少有一次修改，则dump快照
save 900 1
# 300秒（5分钟）之后至少有10次修改，则dump快照
save 300 10
# 60秒（1分钟）之后至少有10000次修改，则dump快照
save 60 10000

# rdb快照信息是否进行压缩
rdbcompression yesr

# rdb快照文件是否损坏，将快照导入到内存中时检查
rdbchecksum yes

# rdb导出的文件名
dbfilename dump.rdb

# rdb导出文件目录
dir ./
```

注：如果不使用rdb快照方式持久化，需要将save开头的三行配置屏蔽掉。

2）利用redis工具redis-benchmark修改数据

在redis的安装目录的bin目录下有一个redis-benchmark工具，可以进行性能测试，包括录入和查询，使用该工具向redis中插入多条数据。

```
# 客户端设置一条数据
set site www.zixue.it

# 服务器端使用redis-benchmark工具模拟大量的数据操作
# 切换到redis的安装目录
# 使用redis-benchmark向redis数据库中执行10000条命令
./bin/redis-benchmark -n 10000

# 查看当前目录下是否dump快照文件dump.rdb
ls ./

# 模拟断电操作，杀死进程
pkill -9 redis

# 再次重启服务，会将快照信息加载到内存中，因此客户端应该还能访问到set变量
./bin/redis-server redis.conf
```

3）问题：rdb方式在保存点间丢失数据

如果客户端在redis服务器dump快照文件后，又进行了其他操作，而系统却在客户端操作后意外的停止了服务，那么再次启动服务后，客户端的操作并没有dump到快照中，因此客户端这部分的操作并没有通过快照被加载到内存中，客户端也就不能访问到dump之后设置的变量信息了。

原因是，dump后客户端操作并没有达到save配置的条件，redis并没有将客户端的操作dump到快照中，也就不能将这部分操作持久化到文件中，再次启动，便不能通过快照加载到系统中，客户端也就不能再访问了。

注：rdb方式的持久化是会在某些情况下出现数据丢失的情况的。

解决方法：配合使用aof持久化方式。

4）rdb工作方式

rdb工作方式包括redis-server工作的主进程和redis-dump进程，当达到了dump条件后，redis-server进程会调用redis-dump进程进行快照的dump。

redis-server会根据配置文件中的配置参数来检查是否到达了导出数据的条件，来决定是否调用redis-dump进程。