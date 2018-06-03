---
title: 微信公共开发平台--阿里云服务器
tags: [weixin]
---

在阿里云中注册并创建服务器实例，作为微信对接的第三方平台系统。

1. 注册并创建服务器实例

创建实例后，可通过公网IP地址来访问服务。在本地使用Putty等远程连接工具即可访问。

注：TCP（22）SSH远程连接Linux实例

2. 安装LAMP运行环境

参考：https://help.aliyun.com/document_detail/50774.html?spm=a2c4g.11186623.6.773.VV0LaS

1）安装apache

a.安装运行apache相关依赖包

```
yum install -y gcc gcc-c++ autoconf libtool
```

b.安装apr

APR（Apache portable Run-time libraries，Apache可移植运行库），主要为上层的应用程序（如Apache）提供一个可以跨越多操作系统平台使用的底层支持接口库。在早期 的Apache版本中，应用程序本身必须能够处理各种具体操作系统平台的细节，并针对不同的平台调用不同的处理函数。

在早期的Apache版本中，应用程序本身必须能够处理各种具体操作系统平台的细节，并针对不同的平台调用不同的处理函数。随着Apache的进一步开发，Apache组织决定将这些通用的函数独立出来并发展成为一个新的项目。这样，APR的开发就从Apache中独立出来，Apache仅仅是使用APR而已。

一般情况下，APR开发包很容易理解为仅仅是一个开发包，不过事实上并不是。目前，完整的APR实际上包含了三个开发包：apr、apr-util以及apr-iconv，每一个开发包分别独立开发，并拥有自己的版本。

apr中包含了一些通用的开发组件，包括mmap，DSO等等

apr-util该目录中也是包含了一些常用的开发组件。这些组件与apr目录下的相比，它们与apache的关系更加密切一些。比如存储段和存储段组，加密等等。

apr-iconv包中的文件主要用于实现iconv编码。目前的大部分编码转换过程都是与本地编码相关的。在进行转换之前必须能够正确地设置本地编码。

```
cd /usr/local/src/
wget http://oss.aliyuncs.com/aliyunecs/onekey/apache/apr-1.5.0.tar.gz
tar zxvf apr-1.5.0.tar.gz
cd apr-1.5.0
./configure --prefix=/usr/local/apr
make && make install
```

c.安装 apr-util

```
cd /usr/local/src/
wget http://oss.aliyuncs.com/aliyunecs/onekey/apache/apr-util-1.5.3.tar.gz
tar zxvf apr-util-1.5.3.tar.gz
cd apr-util-1.5.3
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
make && make install
```

d.安装pcre
 
PCRE（Perl Compatible Regular Expressions）是一个Perl库，包括perl兼容的正则表达式库，apache的安装过程中会依赖该库。

```
cd /usr/local/src/
wget http://zy-res.oss-cn-hangzhou.aliyuncs.com/pcre/pcre-8.38.tar.gz 
tar zxvf pcre-8.38.tar.gz
cd pcre-8.38
./configure --prefix=/usr/local/pcre
make && make install
```

e.安装apache

```
cd /usr/local/src/
wget http://zy-res.oss-cn-hangzhou.aliyuncs.com/apache/httpd-2.4.23.tar.gz 
tar zxvf httpd-2.4.23.tar.gz
cd httpd-2.4.23
./configure \
--prefix=/usr/local/apache --sysconfdir=/etc/httpd \
--enable-so --enable-cgi --enable-rewrite \
--with-zlib --with-pcre=/usr/local/pcre \
--with-apr=/usr/local/apr \
--with-apr-util=/usr/local/apr-util \
--enable-mods-shared=most --enable-mpms-shared=all \
--with-mpm=event
make && make install
```

f.修改httpd.conf配置文件

```
cd /etc/httpd/
vim httpd.conf

# 找到Directory参数，注释掉Require all denied，并添加Require all granted
<Directory />
    AllowOverride none
    # Require all denied
    Require all granted
</Directory>

# 找到 ServerName 参数，添加 ServerName localhost:80
ServerName localhost:80

# 设置 PidFile 路径：在文件最后添加 
PidFile "/var/run/httpd.pid"
```

g.启动服务并访问

```
cd /usr/local/apache/bin/
./apachectl start
netstat -tnlp 

# 使用curl访问apache服务
curl localhost  

# 关闭防火墙
systemctl stop firewalld.service
```

配置80端口放行，实例--管理--安全组--配置规则中添加80访问端口，然后使用客户端机器的浏览器访问服务器的公网IP地址即可。

h.开机启动

```
vim /etc/rc.d/rc.local
# 添加启动命令
/usr/local/apache/bin/apachectl start
```

2）安装mysql

a.检查是否已经安装了mysql

```
rpm -qa | grep mysql

rpm -e 软件名    #注意：这里的软件名必须包含软件的版本信息，如rpm -e mariadb-libs-5.5.52-1.el7.x86_64。一般使用此命令即可卸载成功。
rpm -e --nodeps 软件名   #卸载不成功时使用此命令强制卸载
```

b.安装mysql命令

```
#安装依赖
yum install -y libaio-*                  
mkdir -p /usr/local/mysql
cd /usr/local/src
wget http://zy-res.oss-cn-hangzhou.aliyuncs.com/mysql/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz 
tar -xzvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
mv mysql-5.7.17-linux-glibc2.5-x86_64/* /usr/local/mysql/
```

b.创建mysql相关的用户

```
# 建立 mysql 组和用户，并将 mysql 用户添加到 mysql 组
groupadd mysql
useradd -g mysql -s /sbin/nologin mysql
```

c.初始化数据库

```
# 运行命令初始化 MySQL 数据库
/usr/local/mysql/bin/mysqld --initialize-insecure --datadir=/usr/local/mysql/data/ --user=mysql

# 更改 mysql 安装目录的属性
chown -R mysql:mysql /usr/local/mysql
```

d.设置开机启动以及环境变量

```
cd /usr/local/mysql/support-files/
cp mysql.server  /etc/init.d/mysqld
 # 添加执行权限
chmod +x /etc/init.d/mysqld  

vim /etc/rc.d/rc.local
/etc/init.d/mysqld start

# 设置环境变量
vi /root/.bash_profile
PATH=$PATH:$HOME/bin:/usr/local/mysql/bin:/usr/local/mysql/lib
# 加载环境变量
source /root/.bash_profile
```

f.启动数据库并修改root用户密码

```
# 启动命令
/etc/init.d/mysqld start

# 修改root用户密码
cd /usr/local/mysql/bin
mysqladmin -u root password xxxx
# mysqladmin -u root -p 旧密码 password xxxx

# 登录测试
mysql -uroot -pxxx
```

常见问题

问题1：/usr/local/mysql/bin/mysqld_safe: line 586: /var/lib/mysql/mysqld_safe.pid: No such file or directory ... log-error set to '/var/log/mariadb/mariadb.log', however file don't exists.

原因：mysql启动时无法创建mariadb目录及文件

解决方法：

```
mkdir -p /var/log/mariadb
touch /var/log/mariadb/mariadb.log
chown -R mysql:mysql /var/log/mariadb/
```

问题2：mysqld_safe Directory '/var/lib/mysql' for UNIX socket file don't exists.

解决方法：

```
mkdir -p /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql
```

问题3：Starting MySQL..The server quit without updating PID file 

查看日志文件：/var/log/mariadb/mariadb.log，错误信息会输入到该文件中

```
Fatal error: Can't open and lock privilege  tables: Table 'mysql.user' doesn't exist
```

解决方法：

修改/etc/my.cnf中的数据文件的指向

```
vi /etc/my.cnf
# 指定实际使用的目录
datadir=/usr/local/mysql/data/
```

问题4：error: 'Can't connect to local MySQL server through socket '/tmp/mysql.sock'

原因：根据my.cnf的配置socket=/var/lib/mysql/mysql.sock，mysql在启动后会在/tmp/目录下创建mysql.sock，而mysql.sock配置的是在/var/lib/mysql。因此可以设置一个软连接来解决。

解决方法

```
ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
```

3）编译安装php

a.安装

```
cd /usr/local/src
yum install php-mcrypt libmcrypt libmcrypt-devel  libxml2-devel  openssl-devel  libcurl-devel libjpeg.x86_64 libpng.x86_64 freetype.x86_64 libjpeg-devel.x86_64 libpng-devel.x86_64 freetype-devel.x86_64  libjpeg-turbo-devel   libmcrypt-devel   mysql-devel  -y
wget http://zy-res.oss-cn-hangzhou.aliyuncs.com/php/php-7.0.12.tar.gz
tar zxvf php-7.0.12.tar.gz
cd php-7.0.12
./configure \
--prefix=/usr/local/php \
--enable-mysqlnd \
--with-mysqli=mysqlnd --with-openssl \
--with-pdo-mysql=mysqlnd \
--enable-mbstring \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib --with-libxml-dir=/usr \
--enable-xml  --enable-sockets \
--with-apxs2=/usr/local/apache/bin/apxs \
--with-mcrypt  --with-config-file-path=/etc \
--with-config-file-scan-dir=/etc/php.d \
--enable-maintainer-zts \
--disable-fileinfo
make && make install
```

b.配置文件

```
# 创建配置文件
cd php-7.0.12
cp php.ini-production /etc/php.ini
```

c.在apache中引入php相关模块

```
vim /etc/httpd/httpd.conf

# 在配置文件最后面添加
AddType application/x-httpd-php  .php 
AddType application/x-httpd-php-source  .phps

# DirectoryIndex index.html，添加上对php的首页支持
DirectoryIndex index.php index.html

# 重启apache
/usr/local/apache/bin/apachectl restart
```

d.测试是否能够正常解析PHP

```
cd  /usr/local/apache/htdocs/
# 创建index.php并编辑内容
vim index.php

<?php
phpinfo();
?>

# 重新启动apache
/usr/local/apache/bin/apachectl restart
```

问题：make: *** Zend/zend_execute.lo Error 1

查看日志信息,/var/log/messages，发现 Out of memory: Kill process 3622 (cc1) score 428 or sacrifice child。

解决方法：暂时关闭掉不需要的进程，释放内存

```
# 关闭mysql服务
/etc/init.d/mysqld stop
```

4）安装 phpMyAdmin

```
mkdir -p /usr/local/apache/htdocs/phpmyadmin
cd /usr/local/src/
wget http://oss.aliyuncs.com/aliyunecs/onekey/phpMyAdmin-4.1.8-all-languages.zip
unzip phpMyAdmin-4.1.8-all-languages.zip
mv phpMyAdmin-4.1.8-all-languages/*  /usr/local/apache/htdocs/phpmyadmin
```

访问phpMyAdmin登录页面，输入MySQL的用户名和密码即可登录。