---
title: mysql数据库安装
tags: [database]
---

### 1. 安装mysql

安装版本：mysql-5.6.23-winx64

1）解压mysql，解压后mysql有1.5G大小，设置环境变量path

注：里面有很有调试文件。后缀为.lib或.pdb的，其实可以删除掉

2）创建mysql的配置文件，my.ini文件

```
[mysql]

default-character-set=utf8

[mysqld]

character-set-server=utf8
default-storage-engine=INNODB
max_connections=300

innodb_buffer_pool_size = 128M

basedir = E:\Program Files\mysql-5.6.23-winx64
datadir = E:\Program Files\mysql-5.6.23-winx64\data
port = 3306
```

命令行启动mysql

```
mysqld --defaults-file=../my.ini --user=root
```

3）运行命令查看编码--确认编码设置

```
show variables like 'character%';
```

4）安装mysql服务

cmd定位到mysql的安装目录的bin下，运行命令

```
mysqld --install
```

删除mysql服务: 

```
mysqld --remove
```

5）启动mysql服务

```
net start mysql
```

6）连接mysql

在dos下直接输入mysql，不需要输入密码

7）设置mysql的管理员密码

使用管理员登录：mysql -u root -p，直接回车，不用输入密码即可连接

查看数据库：show databases

为root设置密码：

```
UPDATE mysql.user SET Password = PASSWORD('root') WHERE User = 'root';
FLUSH PRIVILEGES;
```

8）mysql远端地址授权访问

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.13.115' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

注：只允许192.168.13.115机器访问mysql数据库，

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'
```

注：允许所有的主机连接mysql