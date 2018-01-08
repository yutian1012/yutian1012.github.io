---
title: redis主从配置--redis服务器集群
tags: [redis]
---

### 集群的作用

1）从服务器担当主服务器的备份。防止主服务器宕机，可以切换到slave服务器上

2）完成读写分离，主服务器完成写操作，从服务器完成读操作，减轻了主服务器的负担。

3）任务分离，如从服务器可分别分担备份工作与计算工作。

### 集群方式

1）集群方式：

方式一：星型集群，1台master，多台slave（都作为master的从服务器）

```
master <--- slave1
       <--- slave2
       ...
```

方式二：线性集群，1台master，对应1个slave1服务器（做master的从服务器）；slave1服务器也有自己的slave2服务器（做为slave1的从服务器）。

```
master <--- slave1 <--- slave2
```

注：这种方式的好处，master宕机后，可以直接切换到slave1

### 集群原理

master和slave如何同步：

1）从服务器发送同步命令

首先从服务器slave启动后，根据配置发现要与主服务器进行同步，slave从服务器会自动连接到主服务器，并发送sync同步命令；

2）主服务器发送rdb镜像到从服务器

主服务器收到slave发送的同步命令后，主服务器会先dump内存镜像，然后将内存快照rdb发送给从服务器，进行内存镜像的同步操作。

3）主服务器发送aof到从服务器

在从服务器同步rdb的过程中，主服务器可能又接收到了其他的命令，会将这些命令缓冲到aof中。当从服务器同步完成rdb后，主服务器将缓冲在aof中的命令再发送给从服务器，进行同步。

注：完成以上操作后，可实现主服务器与从服务器同步数据。

### 配置集群

1)准备工作：准备3台redis服务器实现redis的集群

```
# 复制redis的配置文件
cp redis.conf redis6380.conf
cp redis.conf redis6381.conf

# 关闭redis的fuw
pkill -9 redis
```

2）修改从服务器的配置文件

执行编辑命令：vim redis6380.conf

```
# 修改从服务器的pid文件
pidfile /var/run/redis6380.pid

# 修改从服务器的port端口
port 6380

# 修改rdb配置
dbfilename dump6380.rdb
dir /var/rdb

# 设置slave参数
# 配置从服务器指向的主服务器，并指定主服务器的端口
# slaveof localhost 6379
# 配置从服务器之都
slave-read-only yes
```

注：从服务器开启rdb，主服务器就可以不开启rdb功能了

注2：打开slaveof选项后才能称为从服务器

注3：从服务器要只读，防止从服务器被写入，造成与主服务器数据不一致问题

同样方式配置另外一台redis从服务器redis6381.conf文件

3）主服务器无须进行更改

注：主服务器可将rdb和aof等持久化功能关闭，持久化任务交由从服务器来处理。

注2：aof可要可不要，如果为了安全可以将主服务器的aof留着，如果主服务器的aof打开，那么从服务器的aof就可以关闭了。

```
# 禁用aof功能
appendonly yes
```

4）启动集群服务

最终配置文件信息：

redis.conf文件rdb关闭，aof打开（主服务器）；redis6380.conf文件中rdb打开，aof关闭（从服务器1）；redis6381.conf文件rdb和aof都关闭（从服务器2）。

```
# 运行主服务器
./bin/redis-server redis.conf
# 运行2台从服务器
./bin/redis-server redis6380.conf
./bin/redis-server redis6381.conf
```

测试连接

```
# 连接主服务器，默认端口是6379
./bin/redis-cli
# 设置变量
set title sunshine

# 启动新客户端，连接从服务器6380端口，看能否读到主服务器设置的title变量
./bin/redis-cli -p 6380
# 查看是否存在title变量
keys 

# 启动新客户端，连接从服务器6381端口，看能否读到主服务器设置的title变量
./bin/redis-cli -p 6381
# 查看是否存在title变量
keys 
```

# redis配置密码

1）主服务器配置密码

```
# 杀掉所有的redis进程
pkill -9 redis

# 修改主服务器配置文件，设置密码为passwd
requirepass passwd

# 启动主服务器
./bin/redis-server redis.conf

# 客户端连接
./bin/redis-cli
# 访问数据前需要配置密码，使用auth命令传递密码（passwd）
auth passwd
# 访问变量，如果在访问变量前不使用auth，会报错
get title
```

2）从服务器需要设置主服务器的密码，否则无法实现集群

编辑从服务器redis6380.conf

```
# 配置主服务器的密码
masterauth passwd
```

注：其他从服务器的配置都是一样的。

### 主从复制的缺陷

每次slave从服务器与主服务器端口后（无论是主动端口，还是网络故障），再次连接主服务器master，都要master全部dump出来rdb进行同步，再同步aof，即同步的过程都要重新执行1遍。

切记多台算slave从服务器不要一下都启动起来，否则master服务器可能IO剧增，甚至可能使master宕机。