---
title: apache虚拟目录
tags: [web服务器]
---

参考：http://blog.csdn.net/klarclm/article/details/9232297

apache服务器使用alias进行配置虚拟目录。

注：在apache的配置文件，配置项一般都是大写字母开头的。

### 什么是虚拟目录

虚拟目录是相对于实际目录来讲的。而这里的虚拟目录和实际目录都是相对于站点的，而访问站点可以使用域名或ip，这里的域名和ip可以简单理解为主机。

1）apache的实际目录如何理解

在apache的配置文件httpd.conf中DocumentRoot属性指定了访问网站的根目录，一般是安装目录下的htdocs目录。那么将html放置到该目录下就能实际访问。

```
# htdocs下新建一个hello.html，将该文件放置到htdocs下
# 访问该文件，只需访问http://localhost/hello.html即可

# 在该站点下创建一个app目录（作为app模块），将hello.html移动到该目录下
# 访问hello.html，只需要访问http://localhost/app/hello.html

# http://localhost/app方式来定位资源的方式，就是使用站点的实际目录来查找资源。并且这里的app是真实存在的目录。
```

localhost可以理解一个域名或IP，理解为站点本身，app理解为一个目录，即相对于站点的目录信息。访问目录下的内容则需要使用相应的url（站点实际目录）进行资源的定位。

2）使用实际目录访问的问题和缺陷

a.探测服务器目录信息

站点的实际目录反应了系统服务器上实际存在的目录信息，很容易被探测到服务器的目录信息。

b.空间限制

实际目录必须位于htdocs目录下，而如果htdocs目录所在的磁盘的空间如果太小，会限制实际目录存放的内容。

3）虚拟目录

虚拟目录可以通过别名的方式来指定一个站点资源实际存放目录。这个别名就可以用来访问站点的实际目录存放的资源，因此有效的隐藏了实际路径信息。另外，虚拟目录指向的路径可以与htdocs（根路径）位于不同的磁盘上，便于空间的扩展。

### 虚拟目录的配置

实例：在系统的E盘下创建一个app文件夹，将该文件夹路径作为站点的虚拟目录，即挂接到站点上，对外提供资源。

1）创建真实的目录

```
# 创建目录
mkdir E:/app
# 创建访问资源hello.html
```

2）设置虚拟目录挂接到站点上

apache使用alias配置虚拟目录。

```
# 编辑apache配置文件httpd.conf，在配置文件的最下面添加
# 定义虚拟目录test，执行实际的物理路径E:/app
Alias /test "E:/app"
# 定义目录访问权限
<Directory "E:/app">
    # 设置选项，禁止显示apache目录列表
    Options Indexes FollowSymLinks
    # 禁止Apache的rewrite模块对URL进行重写，而rewrite规则会写在.htaccess文件里
    AllowOverride None
    # 定义访问规则，会想使用allow规则，再使用deny规则过滤，因此后面的规则起到进一步限制的作用。
    Order allow,deny
    Allow from all
    Require all granted
</Directory>
```

3）重启服务，访问资源

```
# 检查配置文件是否正确
httpd -t
# 重启服务
httpd -k restart
# 访问资源
http://localhost/test/hello.html
```


问题：access访问配置

Invalid command 'Order', perhaps misspelled or defined by a module not included in the server configuration

原因可能是没有加载相应的apache模块，去掉LoadModule access_compat_module注释。

```
LoadModule access_compat_module modules/mod_access_compat.so
```