---
title: memcached的web客户端的使用
tags: [memecached]
---

memadmin工具是有php开发的用于管理memcached的web工具。

1）下载并安装

下载地址：http://www.junopen.com/memadmin/ 

```
# 切换到下载软件目录
cd /home/xxx/download
# 下载软件
wget http://www.junopen.com/memadmin/memadmin-1.0.12.tar.gz
# 解压文件
tar zxvf memadmin-1.0.12.tar.gz
# 将文件解压到ngnix的网站目录下
mv memadmin-1.0.12 /application/nginx/http/xxx

```

2) 为memadmin文件及文件夹分配权限

```
cd /application/nginx/http/xxx
# 分配文件权限为644
find ./memadmin -type f | xargs chmod 644
# 分配文件夹权限为755
find ./emmadmin -type d | xargs chmod 755
# 分配所有者为root
chown -R root.root *
```

3）访问服务，实现memcached服务的后台管理和监控

```
http://xxx/memadmin/index.php
```

![](/images/architecture/memcached/memadmin.png)

4）测试监控

使用linux命令向memcached中存放数据，查看监控信息

```
for n in `seq 10000` ; do printf "set key$n 0 0 10\r\nhelloworld\r\n" nc 127.0.0.1 11211 ; done
```