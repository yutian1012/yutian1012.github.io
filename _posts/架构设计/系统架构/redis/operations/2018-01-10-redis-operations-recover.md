---
title: redis恢复和迁移
tags: [redis]
---

### redis数据恢复

如果redis数据丢失，或者执行了flushall命令，导致数据丢失，如何进行数据的恢复操作。

1）前提：redis服务打开aof持久化

```
[服务器端]
# 启动服务
./bin/redis-server redis.conf

[客户端]
./bin/redis-cli
# 客户端连接并进行操作
set address beijing 
...

# 清空数据库
flushall

# 关闭数据库
shutdown nosave
# 退出客户端
quit

[服务器端]
# 通过aof恢复数据，手动修改aof文件，去掉flushall命令

# 服务器端再次启动，如果有aof和rdb，优先使用aof进行数据载入，这时就能恢复数据
./bin/redis-server redis.conf
```

2）如果aof被重新了怎么办？

如果清空数据库后，aof又被重写了，这时aof内容也被清空了。因此，当数据库被清空了需要立即shutdown，防止aof进行重写。

```
# nosave参数表示shutdown语句不要写入到aof中
shutdown nosave
```

3）如果因为执行了flushall命令导致数据被清空，不能直接使用aof来恢复

因为flushall命令清空了数据库，这时aof文件中有flushall命令，使用aof恢复也会再次执行flushall命令，数据库就又被清空了，因此需要把flushall命令在aof文件中删除。

```
# 编辑文档，将flushall命令及其格式数据删除
vim appendonly.aof
```

### 数据迁移

1）模拟数据迁移，将6379服务器的数据迁移到6380服务器中，迁移过程使用rdb进行。

```
[6379服务器]
./bin/redis-server redis.conf

# 客户端连接，进行数据操作
./bin/redis-cli
# 客户端连接并进行操作
set address beijing 
...

# 将数据dump到6379的rdb文件中
save

# 使用同一台物理服务器，本地测试时需要先shutdown6379的redis服务器
# shutdown nosave

# 拷贝数据到一个新的文件，该文件作为6380服务器的rdb
cp dump.rdb dump6380.rdb

[6380服务器]
# 配置6380服务器，开启rdb功能，并且rdb文档名称为dump6380.rdb，与拷贝的文件一致
# 配置6380禁用aof，防止使用aof进行优先恢复数据
./bin/redis-server redis6380.conf

# 客户端连接，发现不能将数据恢复到6380中
./bin/redis-cli -p 6380

# 停掉6379服务器，然后再次启动6380服务器才能访问到数据。
```

2）问题：文件句柄被占用

迁移过程中，使用rdb迁移数据时，要保证被迁移数据文件的句柄是关闭的。

上面操作的过程中，在拷贝dump.rdb文件时，此时的redis服务器是启动的，因此dump.rdb文件的句柄是被6379服务器占用的，复制出的dump6380.rdb文件的句柄也被该进程占用。这时启动6380服务器，不能够读取到dump6380.rdb文件，因此就不能恢复数据了，因此没有达到数据迁移的目的。

解决方案：

出现此原因是因为我们两个服务器位于同一台机器中，实际过程中应该是2台不同的服务器，因此不会出现文件句柄被占用的问题。

另外，可以通过关闭6379服务器，释放dump6380.rdb文件的句柄，然后再启动6380服务器，重写读取dump6380.rdb，将数据恢复到6380服务器中。

注：linux系统是比较特殊的，在文件打开的同时，可以删除文件。等到打开文件的进程关闭后，该文件所占用的空间才会被释放掉。
