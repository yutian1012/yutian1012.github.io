---
title: redis运维
tags: [redis]
---

1）环境准备：

配置两台redis服务器，端口号分别为6379和6380，并打开redis的rdb和aof。

配置文件分别为redis.conf和redis6380.conf，查看redis6380.conf文件如下

```
pidfile /var/run/redis6380.pid
port 6380

dbfilename dump6380.rdb

appendfilename /var/rdb/appendonly6380.aof

# 都以后台进程运行
daemonize yes
```

2）启动服务

```
# 启动6379服务器
./bin/redis-server redis.conf
```

### 常用运维命令

```
# 客户端连接到redis服务器
./bin/redis-cli

# time命令显示服务器时间，包括2个值：时间戳（秒），微妙数
time

# -------- 数据库的key相关操作---------------
# keys查看所有的key
keys *
# dbsize当前数据库的key的数量
dbsize

# -------- 数据库的持久化相关操作---------------
# 手动调用命令bgrewriteaof重写aof，一般都是满足条件后自动重写
bgrewriteaof
# 手动调用命令save导出rdb快照，会阻塞当前进程
save
# lastsave查看上次导出的rdb的时间
lastsave
# bgsave后台进行导出rdb快照，不会阻塞当前进程
bgsave

# -------- 数据库的清空相关操作---------------
# 清空当前db
flushdb
# 清空所有db
flushall

# info查看redis服务信息
info

# 关闭redis服务
shutdown
```

### info命令查看redis服务信息

info命令可以只返回自己关心的数据

```
# info命令返回redis服务器的所有信息
info

# 如果希望只查看cpu信息，可以使用相关的参数查看
info cpu
```

1）redis中内存的占用情况

```
# info命令能够看到redis内存的占用情况，重点的3个参数
# 实际存放redis数据的内存大小
used_memory:857664
# 为了存放数据，redis申请的内存大小
user_memory_rss:7577600
# 申请的大小与实际使用的比值，比值越小利用率越高
mem_fragmentation_ratio:8.84
```

注：如果mem_fragmentation_ratio比值比较大，说明内存的碎片化比较严重，可以导出再导入，解决内存的碎片

2）主从信息

```
# info命令能够看到redis主从服务信息
# replication
# 当前角色为主服务器
role:master
# 连接的从服务器个数
connected_slaves:1
# 从服务器地址和端口信息
slave0 127.0.0.1,6380,online
```

3）上次dump花费的时间

```
# info命令能够看到redis服务器的状态信息，其中就包括上次dump的处理时间
lastest_fork_usec:2875
```

注：如果lastest_fork_usec使用的时间太长，可能需要想办法优化，因为这样很可能导致redis一直处于dump操作（上一次的dump刚结束，这次的dump操作就开始了），IO读写过于频繁。

### 动态获取/配置redis配置文件的值

```
# 查看配置文件中是否设置了requirepass，即连接服务器是否需要密码
config get requirepass

# 查看配置的慢日志信息
config get slowlog-log-slower-than
# 设置慢日志的时间为100
config set slowlog-log-slower-than 100
```

### 慢查询

```
# 执行多长时间才能称为慢，使用slowlog-log-slower-than参数来决定的，单位是微妙
slowlog-log-slower-than 10000
# 服务器存储多少条慢日志记录，使用slowlog-max-len参数来决定

# 获取慢日志信息
slowlog get
```