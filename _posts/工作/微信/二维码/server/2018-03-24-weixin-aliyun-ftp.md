---
title: 微信公共开发平台--阿里云服务器安装ftp服务
tags: [weixin]
---

参考：https://help.aliyun.com/document_detail/51998.html?spm=a2c4g.11186623.2.8.aLi3jK

1）安装ftp

```
# 安装软件
yum install -y vsftpd
# 查看安装
cd /etc/vsftpd
ls

/etc/vsftpd/vsftpd.conf是核心配置文件
/etc/vsftpd/ftpusers 是黑名单文件，此文件里的用户不允许访问 FTP 服务器
/etc/vsftpd/user_list是白名单文件，是允许访问 FTP 服务器的用户列表

# 安装并设置开机启动
systemctl enable vsftpd.service

# 关闭开机启动
systemctl disable vsftpd.service
```

![](/images/weixin/aliyun/ftp/vsftpd-ls.png)

注：systemctl命令是系统服务管理器指令，它实际上将service和chkconfig这两个命令组合到一起。

2）运行服务

```
# 启动服务
systemctl start vsftpd.service
# 查看服务器的运行
netstat -antup | grep ftp
```

vsftpd安装后默认开启了匿名FTP的功能，使用匿名FTP，用户无需输入用户名密码即可登录FTP服务器，但没有权限修改或上传文件。

3）ftp客户端连接

安装FileZilla进行远程ftp服务器连接测试，端口21。

问题1：FileZilla无法连接到ftp服务器

原因：服务器默认没有开启21端口。

解决方法：

实例--管理--安全组--配置规则中添加21访问端口，优先级与22端口相同（100）。

问题2：服务器发回了不可路由的地址。使用服务器地址代替。

解决方法：更改Filezilla设置，编辑-设置-连接-FTP-被动模式，将“使用服务器的外部ip地址来代替”改为“回到主动模式”即可。

问题3：无法上传文件

原因是：ftp服务器上传文档端口配置为20，因此服务器必须开启20端口的访问权限。

```
# vsftpd.conf文件的配置参数
connect_from_port_20=YES
```

解决方法：开启20端口的访问权限，开启过程类似21的开启方法。

注：连接使用要使用ftp协议，而不是sftp协议

4）配置ftp服务

```
为FTP用户添加写权限，并重新加载配置文件
chmod o+w /var/ftp/pub/
systemctl restart vsftpd.service

# 配置用户
useradd -s /sbin/nologin ftpuser
passwd ftpuser

# 配置vsftpd.conf，禁止匿名用户访问，允许本地用户访问
vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
```

注：anonymous_enable禁止匿名用户，local_enable允许本地用户

5）修改ftp默认的上传路径

vsftpd的匿名的默认根目录为/var/ftp/pub，默认本地用户的路径为自己的home目录

```
chroot_local_user=YES
local_root=/var/www/html

anon_root=/var/www/html
```

注：local_root针对系统用户；anon_root针对匿名用户。