---
title: php应用服务器session共享
tags: [php]
---

参考：https://www.cnblogs.com/kevingrace/p/6031356.html

针对应用服务器的session共享，可以将session信息存放到memcache中，通过启动一个memcache实例来专门存储session信息。该方案适用于java，php等应用服务器集群共享session。

### php如何使用memcache共享session

1）启动一个memcached服务

```
memcached -m 16 -p 11211 -d -c 8192 -u root
```

注：这里的ip地址为10.0.0.9，启动的端口为11211

2）修改php配置文件php.ini

需要修改配置文件的session.save_handler和session.save_path参数

```
vi php.ini

# 默认值为files，即将session信息存储到了文件中
session.save_handler = memcache

session.save_path = "tcp://10.0.0.9:11211"
```

注：需要将所有的php服务都配置到此memcache服务器中。