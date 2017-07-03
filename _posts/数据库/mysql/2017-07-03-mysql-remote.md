---
title: mysql支持远程连接
tags: [database]
---

mysql使用版本5.6

### 通过修改表记录的方式

```
update user set host='192.168.13.14' where host='localhost' and user='root';
```

![](/images/database/mysql/remote/remoteupdateuser.png)

注：host值是安装mysql数据库的允许主机访问的ip地址，安装完成后默认只能自己连接，添加外部的主机地址，如添加192.168.13.72能访问。

```
update user set host='192.168.13.72' where host='192.168.13.14' and user='root';
```

![](/images/database/mysql/remote/remoteupdateuser2.png)

### 2、使用授权方式

实例：

```
mysql> select host from mysql.user where user='root';
+-----------+
| host      |
+-----------+
| 127.0.0.1 |
| ::1       |
| localhost |
+-----------+
```

1）为1192.168.13.115授权远程访问

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.13.115' IDENTIFIED BY 'ipph' WITH GRANT OPTION;

FLUSH PRIVILEGES;
```

注：identified by输入密码

```
mysql> select host from mysql.user where user='root';
+----------------+
| host           |
+----------------+
| 127.0.0.1      |
| 192.168.13.115 |
| ::1            |
| localhost      |
+----------------+
```

注：可以看到运行完成后增加了一个远程地址。

2）如果对所有的远程用户使用%代替。

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'
```

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
Query OK, 0 rows affected

mysql> select host from mysql.user where user='root';
+----------------+
| host           |
+----------------+
| %              |
| 127.0.0.1      |
| 192.168.13.115 |
| ::1            |
| localhost      |
+----------------+
```

3）撤销授权：

```
revoke all on *.* from 用户@’192.168.13.115’;
```

注：root不能撤销，只能在mysql.user表中删除对应的host记录。

```
delete from mysql.user where host='192.168.13.115';
```

### 防火墙显示访问

mysql数据库连接不上还可能是因为服务器端的防火墙拦截了3306端口。需要防火墙去掉对3306端口的限制。

控制面板---Windows防火墙--打开或关闭防火墙。
