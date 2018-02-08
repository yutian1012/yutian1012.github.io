---
title: apache虚拟主机
tags: [web服务器]
---

Apache服务器上可以配置多个虚拟主机，实现一个服务器提供多站点服务。

### 什么是虚拟主机

访问web服务，本质上看（从协议上）是访问某个IP的主机上的某个端口（默认是80）通常需要通过访问不同的域名或者端口实现对不同网站的访问（具体到服务器里就是不同目录），这个时候就需要设置虚拟主机（VirtualHost）。

1）如何理解虚拟主机

搭建完成apache服务器后，可以使用localhost来访问主机。同时可以htdocs中定义目录或使用虚拟目录来存放资源。在访问时都会需要使用localhost（默认80端口）。

```
# 只有path部分是可以变动的，localhost是不能变的
http://localhost/{path}
```

虚拟主机的目的就是可以改变前面的localhost而引入的，或者改变端口，不使用默认的80端口。

```
# 配置hello.com，world.com来定义虚拟主机。
# 对外看来像是不同的服务器，其实使用的是同一个服务器，配置了不同的虚拟主机。可以通过ping命令发现ip地址相同的方式来确认。
http://hello.com
http://world.com
```

2）虚拟主机的三种模式

虚拟主机分为三种模式，基于域名、基于IP地址、基于端口号，常用的为基于域名和IP地址。

参考：http://blog.51cto.com/sunjie123/1735865

### 基于端口的虚拟主机

1）创建两个网站，网站1目录设置为E:/web1，网站2目录设置为E:/web2

```
# 创建站点目录
mkdir E:/web1
mkdir E:/web2

# 修改配置文件httpd，在文件末尾添加配置
# 使用Include命令将相关配置信息加载到主配置文件中
Include conf.d/*.conf

# 在apache安装目录下创建conf.d目录，与conf目录同级，并放置相关的配置文件
mkdir %apache_home%/conf.d

```

注：或者打开主配置文件httpd中的Include conf/extra/httpd-vhosts.conf配置信息。

2）子配置文件监听81端口

```
# 在conf.d目录下，创建子配置文件vhost.conf

------------------------------------------------------------------
# 监听端口
Listen 81
# web1目录的访问权限设置
<Directory "E:/web1">        
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>

# 指定虚拟主机的IP地址和端口号，这里的地址是本地的ip地址
<VirtualHost *:81>
    # ServerAdmin是指在服务器有任何问题将发信到这个地址
    # ServerAdmin webmaster@dummy-host.example.com
    # 指定web文件目录
    DocumentRoot E:/web1
    # 域名
    ServerName 192.168.1.3
    # 错误日志
    ErrorLog logs/web1.com-error_log
    # 访问日志
    CustomLog logs/web1.com-access_log common
</VirtualHost>
------------------------------------------------------------------

# 检查配置信息，并且启动测试
httpd -t
# 重启服务
httpd -k restart
# 访问服务，不能使用localhost:81，这样访问的还是htdocs下的资源
# 访问地址192.16.3.1.3:81，这样此能访问到E:/web1目录下定义的资源
http://192.16.3.1.3:81
```

注：也可以将Directory配置到VirtualHost标签中，则不需要再指定DocumentRoot

3）子配置文件监听82端口

```
# 编辑conf.d/vhost.conf文件，配置82端口
Listen 82
<Directory "E:/web2">        
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>

<VirtualHost *:82>
    DocumentRoot E:/web2
    ServerName 192.168.1.3
    ErrorLog logs/web2.com-error_log
    CustomLog logs/web2.com-access_log common
</VirtualHost>
```

注：主配置文件httpd.conf加上vhost.conf后，服务器监听80，81，82。其中localhost，localhost:81，localhost:82都能跳转到htdocs下。

### 基于域名配置虚拟主机

1）修改conf.d/vhost.conf文件

```
<Directory "E:/web1">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>

# 这个地方不要在给端口号了
<VirtualHost *:80>
    DocumentRoot E:/web1
    # ServerName配置域名
    ServerName www.web1.com
    ErrorLog logs/web1.com-error_log
    CustomLog logs/web1.com-access_log common
</VirtualHost>
```

2）修改hosts文件

```
192.168.1.3 www.web1.com
```

3）配置web2

```
<Directory "E:/web2">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>

# 这个地方不要在给端口号了
<VirtualHost *:80>
    DocumentRoot E:/web2
    # ServerName配置域名
    ServerName www.web2.com
    ErrorLog logs/web2.com-error_log
    CustomLog logs/web2.com-access_log common
</VirtualHost>
```

### 基于ip配置虚拟主机

主要针对服务器存在多个网卡，对不同网卡地址进行绑定不同站点。

如系统有无线网卡和有线网卡。其中无线网的ip地址为192.168.1.3，有线网卡地址为192.168.1.9。apache的根目录会占用一个ip地址，作为自己服务器地址。而虚拟机只能使用剩下的一个ip地址。因此需要再虚拟出一个网卡，地址为192.168.1.20。

注：根目录一般是从ip地址小的开始适配，这里会适配129.168.1.3。

注2：在linux系统中配置一个虚拟ip地址更为简单。

1）修改conf.d/vhost.conf文件

```
# 配置无线网卡的虚拟主机服务器
<Directory "E:/web1">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>

# 修改此处网卡IP地址即可
<VirtualHost 192.168.1.9:80>
    DocumentRoot E:/web1
    ErrorLog logs/web1.com-error_log
    CustomLog logs/web1.com-access_log common
</VirtualHost>

# 配置有线网卡的虚拟主机服务器
<Directory "E:/web2">
    Options Indexes MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>

# 修改此处网卡IP地址即可
<VirtualHost 192.168.1.20:80>
    DocumentRoot E:/web2
    ErrorLog logs/web2.com-error_log
    CustomLog logs/web2.com-access_log common
</VirtualHost>
```

2）windows配置虚拟网卡

参考：https://jingyan.baidu.com/article/b87fe19eb3f0f05219356858.html

设备管理--操作（菜单）--添加过时硬件，选项设置：安装我手动从列表选择的硬件（选择网络适配器）--Microsoft（Microsoft loopback adapter）.

![](/images/web/apache/windows-virtual1.png)

![](/images/web/apache/windows-virtual2.png)

![](/images/web/apache/windows-virtual3.png)

![](/images/web/apache/windows-virtual4.png)

注：设置完成后会在网络连接中看到一个新设置的本地连接。

3）手动设置新生成的本地连接的ip地址为192.168.1.20

![](/images/web/apache/windows-virtual5.png)