---
title: redis应用（七）--ab测试
tags: [redis]
---

使用linux中ab进行性能测试。ab功能是apache提供的，即在httpd的bin目录下存在该工具。

1）新建提供的数据文件

```
vim /tmp/weibopost

# 设置提交的内容
content=test
```

2）提交测试数据

```
cd /usr/local/httpd/bin

# 执行ab命令发送请求，-c表示一个客户端，-n表示发送的请求次数，
# -H添加header头信息，-p指定post提供的数据
./ab -c 1 -n 2 -H 'Cookie:username=ssss;userid=23;authsecret=xxxxx' -p /tmp/weibopost -T 'application/x-www-form-urlencoded'
```