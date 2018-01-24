---
title: yum源信息
tags: [linux]
---

有时用yum安装软件时比较慢，可以通过手动调整yum源的方式，将镜像地址改为中国的镜像地址。

1）使用阿里云的镜像

地址：http://mirrors.aliyun.com/help/centos

```
# 运行命令会自动将yum源调整为阿里云镜像
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```