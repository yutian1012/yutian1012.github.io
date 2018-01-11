---
title: redis监控工具
tags: [redis]
---

实验环境：星型集群，1台master，多台slave（都作为master的从服务器）

```
# 主服务器配置文件，端口6379
redis.conf
# 两从服务器配置文件
redis6380.conf
redis6381.conf
```

注：从服务器的配置参考之前的主从服务器配置。

1）问题：如果主服务器宕机怎么办？

需要将其中1台从服务器变成主服务器（6380），而另一台从服务器执行新的主服务器（6381）。

动态更改服务器主从状态

```
[6379]
# 连接主服务器6379
./bin/redis-cli
# 使用命令关闭主服务器，模拟主服务器宕机
shutdown

[6380]
# 连接从服务器6380
./bin/redis-cli -p 6380
# 查看主从信息，可以看到状态master_link_status:down
info Replication

# 将6380服务器变成主服务器
# 修改该服务器使其不再是从服务器
slaveof no one
# 修改readonly为yes，使其可写
config set slave-read-only no

[6381]
# 将6381服务器做为6380服务器的从服务器
slaveof localhost 6380
```

### sentinel工具监控主从服务器

sentinel服务是使用redis-server来启动的。

1）原理：

sentinel会不断和master进行通信，检测心跳，一旦发现master服务器不响应了，则自动切换从服务器为主服务器，并更改其他从服务器的指向。

2）sentinel监控配置

```
# 将sentinel.conf文件拷贝到redis的安装目录
cp /usr/local/src/redis-2.6.16/sentinel.conf ./

vim sentinel.conf

# 定义监控主机和端口
# def_master是自定义名称，所有的配置要使用相同的名称
#最后的参数2表示连续多少个监控服务（哨兵）没有连接上则认为连接失败
# 这里设置为1，我们只配置一个sentinel服务（哨兵）
sentinel monitor def_mater 127.0.0.1 6379 1

# 设置授权密码，如果主服务器设置了密码则需要在sentinel中设置
sentinel auth-pass def_mater xxxxx

# 设置sentinel多长时间没有收到回应，认为服务器宕机，单位毫秒
sentinel down-after-millseconds def_master 30000

# 是否允许切换其他从服务器为主服务器，
#可能存在多个sentinel监控服务在监控主服务器，最终修改只能由一个监控服务sentinel来修改，因此需要设置该选项，目的是保证只有一个sentinel能够修改，而其他sentinel不能修改
sentinel can-failover def_master yes

# 同时把多少台slave指到新的主服务器上，指定1次只能有1台连接到主服务器
sentinel parallel-syncs def_master 1

# 15分钟如果没有完成主从的顺利切换，则会发生
sentinel failover-timeout def_master 900000

当failover时，可以指定一个通知脚本用来告知系统管理员
sentinel notification-script def_master /var/redis/notify.sh
```

3）问题：防止一次性将所有的slave连接到新的主服务器

如果有6台主从服务器，当主服务器宕机后，sentinel将1台从服务器提升为主服务器，这时其他4台从服务器修改指向，指向新的主服务器。每次slave断开再连接时都要master全部dump出来rdb，再aof，即同步过程要重复执行多次。造成master的IO操作剧增。

注：多台从服务器不要一下都切换到新的主服务器，可能造成新的主服务器IO剧增。

4）sentinel自动切换主服务器测试

```
# 关闭所有的redis进程
pkill -9 redis

# 将三台服务器启动起来，6379做主服务器
./bin/redis-server redis.conf
./bin/redis-server redis6380.conf
./bin/redis-server redis6381.conf

# ./bin/redis-server --help查看帮助，如何启动sentinel
# 启动sentinel进程
./bin/redis-server sentinel.conf --sentinel

# 关闭6379服务器
./bin/redis-cli
# 执行关闭命令
shutdown
```

5）问题：sentinel如何选择哪个从服务器做主服务器

sentinel在未设置的情况下，会随机的在从服务器中选定一台做为主服务器，不便于程序的编写和调试，可能python中的切换连接顺序是一定的，而这里主服务器却随机选择的，造成写操作冲突。

解决方法，在从服务器中配置slave的优先级。

```
vim redis6380.conf
# 设置slave的优先级，数值越小优先级越高
slave-priority 10
```