---
title: nginx配置
tags: [nginx]
---

nginx的配置文件是nginx.conf，该文件位于安装路径下。

1）配置文件的server项：

```
server {
    # 监听端口
    listen       80;
    # 域名，空格用于区分多个域名，www是子域名，一般用于万维网，bbs等用于论坛
    server_name  www.abc.com abc.com;

    # 将请求转发到tomcat服务器上
    location / {
        #本地访问URL
        proxy_pass   http://127.0.0.1:8080/;
    }

    # 定义静态文件查找路径，将tomcat的帮助文档定位到doc下
    location /doc/ {
        alias D:/Program Files/apache-tomcat-7.0.59/webapps/docs;
    }

    # 配置服务器错误页面，root指定html是ngnix目录下的html，是相对目录
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root html;
    }
}
```

注：location节点中的元素后必须跟分号，否则报错，多个值之间使用空格分隔。

注2：proxy_pass表示转发，root指定路径。

2）server_name指定域名

在hosts文件中配置域名，文件位置：C:\Windows\System32\drivers\etc\hosts

```
127.0.0.1 www.abc.com
```

注：本地配置域名测试

3）location定位服务或资源目录

配置tomcat的代理转发功能

```
# 将请求转发到tomcat服务器上
location / {
    #本地访问URL
    proxy_pass   http://127.0.0.1:8080/;
}
```

注：这里的8080是tomcat服务启动的端口

启动tomcat服务，访问tomcat可通过ngnix来实现，访问：http://www.abc.com

4）配置静态资源，静态在于可单独放置到一个静态资源服务上

```
# 定义静态文件查找路径，将tomcat的帮助文档定位到doc下
location /doc/ {
    alias D:/Program Files/apache-tomcat-7.0.59/webapps/docs;
}
```

可将图片等信息放置到系统的某个文件夹下或相应的静态服务器上。

访问资源时访问：http://www.abc.com/xxx.js

5）全局的错误信息

```
# 配置服务器错误页面，root指定html是ngnix目录下的html，是相对目录
error_page 404  /404.html
error_page 500 502 503 504 /50x.html;
# 50x.html到站点根目录下的html文件夹中查找
location = /50x.html {
    root html;
}
```

注：100，502等errorcode都映射到/50x.html错误页面上

6）访问

访问域名abc.com会跳转到tomcat启动目录下

```
http://www.abc.com/
```

访问/doc目录下的静态文件

```
http://www.abc.com/doc
```

访问tomcat下的其他服务（位于tomcat/webapps目录下配置的其他服务）

```
http://www.abc.com/xxx
```