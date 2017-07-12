---
title: mysql数据库安装（5.7版本zip包）问题
tags: [database]
---

### 丢失msvcr120.dll

安装平台windows server 2008 R2 Standard，使用zip解压版安装mysql5.7.18，在执行初始化命令是弹出错误提示信息。

```
bin\mysqld --initialize-insecure
```

![](/images/database/mysql/install/msvcr120dlllost.png)

解决方案：

需要安装VC++2013x64version，而如果使用installer方式安装，则会自动安装该环境。安装该c++运行库可到微软的官网上下载安装。

### 配置远程

配置mysql远程访问，运行grant命令错误

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
1133 - Can't find any matching row in the user table
```

参考：https://dev.mysql.com/doc/refman/5.7/en/adding-users.html

```
CREATE USER 'root'@'%' IDENTIFIED BY 'some_pass';

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

flush privileges;
```