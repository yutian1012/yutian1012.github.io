---
title: memcached安装
tags: [memecached]
---

memcached没有相应的安全机制，连接上就可以读写，因此可以通过-l参数指定监听内网ip来实现安全控制。

1）查看linux版本

```
# 查看版本信息
uname -r
```

2）安装前准备

安装memcached前需要先安装libevent，以源码的方式安装libevent

```
# 将下载的软件放置到一个位置
mkdir /home/xxx/software
# 切换目录
cd /home/xxx/software
# 下载源码安装
wget http://www.monkey.org/~provos/libevent-1.4.13-stable.tar.gz
# 解压
tar zxf libevent-1.4.13-stable.tar.gz
# 进入解压目录
cd libevent-1.4.13-stable
# configure软件
./configure
# 安装
make & make install
```

注：查看configure参数信息使用./configure --help

3）安装memcached服务端

```
# 切换目录
cd /home/xxx/software
# 下载软件
wget http://memcached.org/files/old/memcached-1.4.13.tar.gz
# 解压
tar zxf memcached-1.4.13.tar.gz
# 切换目录
cd memcached-1.4.13
# configure软件
./configure
# 安装
make & make install
```

4）启动服务

```
# 查看安装到的目录信息，找到安装目录为
which memcached
# 启动，-m指定分配给memcached的内存大小，-p指定端口号，默认是11211，
# -d表示以damon后台服务的方式运行，-c并发的数量，默认是1024，-u指定启动用户 
# -l指定监听的ip地址（一般使用内外ip，安全策略）
memcached -m 16 -p 11211 -d -c 8192 -u root
# 查看服务是否启动
netstat -lntup| grep memcached

# 再次启动一个memcached实例，只需要修改一个端口即可
memcached -m 16 -p 11212 -d -c 8192 -u root
# 查看两个memcached服务进程
ps -ef|grep memcached
```

5）关闭memcached

```
# 查看进程
ps -ef | grep memcached
# 方式一：使用pid进程号关闭--pid通过查询获取
kill -9 123456

# 方式二：使用pid关闭--pid在文件中获取
# 启动时指定pid文件
memcached -m 16 -p 11211 -d -c 8192 -u root -P /var/run/m1.pid
# 查看pid文件中的pid内容，值为18670
cat /var/run/m1.pid
# 杀掉进程
kill -9 `cat /var/run/m1.pid`

# 方式三：将memcached进程全部杀掉
pkill memcached
```

6）问题

启动memcached服务时报错，libevent-1.4.so.2:cannot open shared object file.

解决方法：

```
# 查找安装的libevent库的路径，这里安装到了/usr/local/lib/libevent-1.4.so.2
find /-name "libevent-1.4.so.2"
# 将路径设置到/ect/ld.so.conf文件中
# 该文件记录想要读入高速缓存中的动态函数库所在的目录，是目录不是文件
vi /etc/ld.so.conf
# 将路径追加到该文件中
/usr/local/lib/
# 利用ldconfig执行文件将 /etc/ld.so.conf的数据读入高速缓存中
idconfig
```