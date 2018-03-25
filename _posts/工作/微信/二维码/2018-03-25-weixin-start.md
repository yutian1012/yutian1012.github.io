---
title: 微信入门开发流程
tags: [weixin]
---

参考：http://www.cnblogs.com/txw1958/p/wechat-tutorial.html

1）公网可访问的服务器

这里使用的是阿里云服务器，并配置好lamp环境。

```
# 配置安装vsftpd，并启动，用于上传程序
systemctl restart vsftpd.service
# 安装apache，并启动
/usr/local/apache/bin/apachectl start
# 安装mysql，并启动
/etc/init.d/mysqld start
# 安装php并配置apache的php模块
```

2）申请微信开发者账号

在微信公众平台注册上注册，主要用于管理公众号。由于申请过程比较繁琐，这里使用微信提供的测试账号进行开发测试。

```
微信号:gh_2626e3fe4187 
appID:wx1552c70aa6782a58
appsecret:e6095963fb618886a32305af2d90cfe1

url:http://47.104.31.92
token:searchpatent
```

注：在测试账号认证通过前需要在服务器端上传认证代码进行认证，否则无法使用。验证URL有效性成功后即接入生效，成为开发者。

3）编辑程序并上传到服务器

```
# 通过ftp客户端FileZilla上传php文件
# 复制到apache服务目录下
cp /home/ftpuser/index.php /usr/local/apache/htdocs/
```

4）测试消息发送

access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。可通过接口获取access_token。另外发送消息要事先知道用户的openID，这个可以通过用户关注账号事件时获取并存储。微信用户的openID在该公众号上是唯一的。

接口测试工具：https://mp.weixin.qq.com/debug/

![](/images/weixin/develop/mp/send-message.png)