---
title: mysql数据库安装（5.7版本zip包）
tags: [database]
---

5.7版本与之前的版本最主要的区别是需要手动运行命令初始化数据库。

参考：https://dev.mysql.com/doc/refman/5.7/en/windows-install-archive.html

### 数据库初始化

1）首先创建一个mysql的配置文件my.ini

参考：https://dev.mysql.com/doc/refman/5.7/en/windows-create-option-file.html

```
[mysqld]
# set basedir to your installation path
basedir=E:/mysql
# set datadir to the location of your data directory
datadir=E:/mydata/data
```

注：假设mysql的解压目录为E:/mysql

2）初始化数据

5.7版本的数据库不再提供data以及相应的库文件。需要在使用时提前初始数据

参考：https://dev.mysql.com/doc/refman/5.7/en/data-directory-initialization-mysqld.html

注：此时在mysql安装目录下不存在data文件夹。

在cmd中将目录定位到mysql的安装目录（不是定位到bin下），运行命令

```
bin\mysqld --initialize
或者 bin\mysqld --initialize-insecure
```

注：如果成功运行，会在mysql的安装目录下出现data目录，并在data目录下找到一些数据库信息。

### 数据库root的密码设置

如果初始化数据时使用的是--initialize参数，则运行命令可设置一个随机密码

```
mysql -u root -p
Enter password: (enter the random root password here)
```

如果初始化数据时使用的是--initialize-insecure参数，则运行命令

```
mysql -u root --skip-password
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

### 字符集设置

参考：https://dev.mysql.com/doc/refman/5.7/en/charset.html

### 数据sqlmodel的配置

1）日期格式不支持'0000-00-00 00:00:00'

在配置文件中my.ini中添加sqlmodel的配置

```
NO_ZERO_IN_DATE,NO_ZERO_DATE
```

2）增强了group by的校验

原来select中的很多字段可以不在group by中出现，5.7版本加强了这方面的校验。可以打开该功能，也可以关闭。需要在sql_model中设置。

```
ONLY_FULL_GROUP_BY
```

3）修改sql_model配置

```
修改前：
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
修改后：
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

注：修改后系统可以支持日期格式'0000-00-00 00:00:00'，并且对groupby进行了增强