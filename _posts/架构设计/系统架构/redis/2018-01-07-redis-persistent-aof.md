---
title: redis持久化aof
tags: [redis]
---

aof是日志方式的持久化。日志是指做了什么操作都会记录下来，可以通过日志文件信息进行文件系统的恢复操作。

1）配置文件

```
# 打开aof功能
appendonly yes
# aof文件名 
appendfilename ./appendonly.aof
# aof的三种同步方式，选择一种开启
# 每个命令都立即同步到aof中，这种方式安全，但速度慢
appendfsync always 
# 折中方案，每秒写一次aof
appendfsync everysec
# 写入工作交由操作系统，由操作系统判断缓冲区大小，统一写入到aof
appendfsync no

# 正在导出rdb快照的过程中，要不要停止同步aof
no-appendfsync-no-rewrite yes

# 文件大小比起上次重写时的大小，增长率100%时，重写，如上次重写时是2M，这次增加到了4M，则重写
auto-aof-rewrite-percentage 100
# aof文件至少超过64M时，重写
auto-aof-rewrite-min-size 64mb
```

注：开始时aof文件大小很小，增长的比较快，则前期aof重写频率会非常频繁，因此可以通过配合auto-aof-rewrite-min-size参数加以限制。

注2：aof重写的两个参数同时满足时才会发生aof重写。

注3：aof重写会改变aof文档的大小的，如重写前aof已经3M，但重写后aof文档却可能变成3K大小。

2）aof原理

redis-server会接收客户端的命令并执行，这时会调用redis-aof子进程记录客户端发送的命令，redis-aof子进程会将信息记录到文本文件中，作为日志文件。

3）慢的问题，aof记录数据会比内存记录缓慢的多

aof是向磁盘文件中写入数据，而redis-server是直接操作内存的，两者之间存在速率等级上的问题。

解决方案：在写aof时并不是逐条命令的写入，而是按照一定的策略写入，如没秒执行一次。

4）操作过程

测试aof文档记录命令：

开启aof功能，使用客户端连接服务器，然后向redis中设置数据，查看aof文当，可以看到客户端设置的所有命令此时都记录在了aof文档中。

测试aof重写功能

使用redis-benchmark发生大量命令来测试aof的重写功能

```
./bin/redis-benchmark -n 20000
```

手动测试重写：可以使用命令来命令redis进行重写

```
# 使用命令来重写aof
begrewriteaof
```

### 常见问题

1）在dump rdb过程中，aof如果停止同步，会不会丢失？

不会丢失，所有的操作缓存在内存的队列里，dump完成后统一进行aof操作。

2）关掉redis后又重启，但rdb文件没有发挥作用？

rdb文件有内容，开启了aof选项，重启redis，产生了1个空的aof文件。此时aof和rdb都存在，会优先以aof来恢复持久数据，因此redis的内容空了，而rdb没有起作用。

3）aof记录每条命令，若针对同一个变量，执行100次加1操作，恢复时也执行100次操作才能恢复，这些效率会很低，如何解决？

分析：内存中的key在操作过后都会有一个位置的值，如果把内存的key/value逆化成相关的命令，记录到aof中，假设age变量指向100次加1操作，最终值为101，那么可不可以直接将set age 100记录到aof中，而不是将100次的加1操作记录到aof中。

解决方案：可以使用aof的重写来实现。

对重写的理解：财务中会记录一年中员工借出的每笔款项，而员工实际想要知道这一年中最终借出了多少钱（一个总数，如果想要查找具体信息，再看每笔款项的记录）。而这个总数就可以理解为aof的重写功能，即对内存变量最终信息的逆命令化并记录到aof文件中。