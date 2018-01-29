---
title: memcached的监控
tags: [memecached]
---

### memcached的状态

1）linux的shell中直接运行查看

```
# 查看所有的状态
printf "stats\r\n" |nc 127.0.0.1 11212
# 查看设置信息
printf "stats settings\r\n" |nc 127.0.0.1 11212
# 查看内存区块slabs的信息
printf "stats slabs\r\n" |nc 127.0.0.1 11212
# 查看数据项统计，存放的对象数据信息
printf "stats items\r\n" |nc 127.0.0.1 11212
# 查看存在的对象Item个数和大小
printf "stats size\r\n" |nc 127.0.0.1 11212
# 清理统计数据
stats reset
```

2）统计命令stats（显示所有内容）

![](/images/architecture/memcached/stats.png)

a.分析cpu占用是否高

```
rusage_user     --该进程累计的用户时间，单位秒
rusage_system   --该进程累计的系统时间，单位秒
```

b.分析连接数是否太多

```
curr_connections    --当前连接数量
total_connections   --memcached运行依赖接受的连接总数
```

c.分析命中率是否太低

```
cmd_get     --查询请求总数
get_hits    --查询成功获取数据的总次数，命中次数
get_misses  --查询未获取到数据的总次数，未命中次数
```

d.分析字节数流量

```
bytes           --memcahced当前存储内容所占用字节数
bytes_read      --memecached从网络读取到的总字节数
bytes_written   --memcached向网络发送的总字节数
```

e.分析对象数LRU频率

```
curr_items      --memcached当前存储的内容数量
total_items     --memcached启动以来存储过的内容总数
evictions       --LRU释放对象数，用来释放内存
```

3）stats settings命令查看设置信息（只显示settings的信息）

![](/images/architecture/memcached/stats-settings.png)

主要参数信息

```
maxbytes            --最大字节数限制（0无限制）
maxconns            --允许最大连接数
tcpport             --TCP端口
updport             --UDP端口 
oldest              --对象的最大过期时间 
evictions           --on/off，是否禁用LRU
growth_factor       --增长因子
chunk_size          --key+value+flags大小
num_threads         --线程数，可以通过-t设置
```

4）stats items命令，查看存储信息

```
number          --slab中对象数，不包含过去对象
evicted         --LRU释放对象数
evicted_time    --最后剔除的这个数据已经被缓存的时间，监控频率
outofmemory     --不能存储对象次数，使用-M会报错
```

5）区块统计

![](/images/architecture/memcached/stats-slabs.png)

```
chunk_size          --chunk大小，单位字节
chunks_per_page     --每个page的chunk数量
total_pages         --page数量
total_chunks        --总的chunk数量

get_hits            --get命中数
cmd_set             --set数量

used_chunks         --已被分配的chunk数
free_chunks         --剩余chunk数
free_chunks_end     --分完page浪费chunk数
mem_requested       --请求存储的字节数（数据存储的字节数）

active_slabs        --slab数量
total_malloced      --总内存数量

浪费内存字节数 = (used_chunks * chunk_size) -mem_requested
注：如果浪费空间太大，需要调整factor
```

### 利用脚本监控me mcached

参考（nagios监控工具）：http://blog.51cto.com/storysky/244962

1）监控存活

可以通过监控进程或端口来监控是否存活，使用脚本来监控服务是否启动，脚本可以使用shell或php等

2）监控命中

命中的概率低时需要调整策略。

3）监控响应时间

