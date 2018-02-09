---
title: apache访问权限设置
tags: [web服务器]
---

参考：https://www.cnblogs.com/top5/archive/2009/09/22/1571709.html

Allow和Deny可以用于apache的conf文件或者.htaccess文件中（配合Directory, Location, Files等），用来控制目录和文件的访问授权。

注：apache的访问授权的应用规则与firewall的应用规则是不同的，理解时不要混淆。

1）常用配置

```
# deny没定义，表示拒绝所有，但是允许所有
# 所有地址都能访问
Order deny,allow
Allow from All
```

先设定"先检查禁止设定，没有禁止的全部允许"，而第二句没有Deny，也就是没有禁止访问的设定，直接就是允许所有访问了。

2）最后规则决定作为但是使用

```
# 运行所有，但拒绝ip1和ip2访问
# ip1和ip2不能访问
Order allow,deny
Allow from all
Deny from ip1 ip2
```

3）错误配置允许所有

```
# 拒绝domain.org域名，但允许所有
# 所有地址都能访问
Order deny,allow
Allow from all
Deny from domain.org
```

4）错误配置拒绝所有

```
# 运行ip1方法，但拒绝所有
# 所有地址都不能访问
Order allow,deny
Allow from ip1
Deny from all
```

解决方法：

```
# 拒绝所有，但运行ip1访问
# 只有ip1能访问
Order Deny,Allow
Deny from all
Allow from ip1
```

注：这种方式适合只允许指定ip访问资源。

5）总结，影响最终判断结果的只有两点：

a.order语句中allow、deny的先后顺序；

b.allow、deny语句中各自包含的范围。

注：将最后的规则理解为但是。

6）Apache有一条缺省规则（Allow和Deny语句都不在单独配置）

"order allow,deny"本身就默认了拒绝所有的意思，因为deny在allow的后面。

"order deny,allow"本身默认的是允许所有。

### 实例

本地ip地址192.168.1.3，配置了一个虚拟目录，并对该目录进行访问权限设定

1）创建待访问的虚拟目录

```
Alias /test "E:/app"
<Directory "E:/app">
    Options Indexes FollowSymLinks
    AllowOverride None
    ....
    Require all granted
</Directory>
```

2）限定实例1

```
# 拒绝所有，但运行192.168.1.3，结果192.168.1.3能够正常访问虚拟目录
Order deny,allow
Allow from 192.168.1.3
Deny from all
```

3）限定实例2

```
# 交换律allow和deny的顺序
# 允许192.168.1.3，但拒绝所有，结果所有的路径都不能访问
Order allow,deny
Allow from 192.168.1.3
Deny from all
```

注：在理解权限时，理解为但是。首先根据order设定的顺序限定，然后使用但是来排除。

### 图形的方式理解

1）实例1

```
Order allow,deny
Allow from example.com
Deny from bbs.example.com
```

只允许example.com域名，但排除bbs.example.com域名，访问资源。所有不属于example.com域名的都不允许访问（应用最后的deny规则）。

![](/images/web/apache/appache-access1.png)

2）实例2

```
Order deny allow
Deny from all
Allow from example.com
```

拒绝所有的地址访问，但允许example.com域名地址访问。由于deny包含了所有，因此没有其他地址再应用allow规则来被允许了。

![](/images/web/apache/appache-access2.png)