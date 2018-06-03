---
title: linux中修改主机名
tags: [linux]
---

1）查看主机名命令

```
# 查看机器名
hostname   
# 查看本机器名对应的ip地址
#hostname -i 
```

2）修改主机名

修改/etc/sysconfig/network文件（修改主机名称）

```
vi /etc/sysconfig/network
HOSTNAME=yutian
```

3）修改访问域名对应的ip

修改/etc/hosts文件（修改访问域名对应的ip，只有本地访问这些域名时，才能映射成相应的ip）

```
vi /etc/hosts
127.0.0.1 eureka7001.com eureka7002.com eureka7002.com

# 测试域名设置是否成功
ping eureka7001.com
```

注：使用空格分隔多个域名