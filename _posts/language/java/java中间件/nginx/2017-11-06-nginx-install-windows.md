---
title: nginx的安装使用--windows
tags: [nginx]
---

参考：http://www.cnblogs.com/saysmy/p/6609796.html

Nginx是一个web服务器也可以用来做负载均衡及反向代理使用，目前使用最多的就是负载均衡。

1）下载

下载地址：http://nginx.org/en/download.html

2）安装

安装过程：解压到本地即可。解压压缩包，并配置nginx.conf文件。

新建server项：

```
server {
    #监听端口
    listen       80;
    #域名
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

3）测试

在hosts文件中配置域名，文件位置：C:\Windows\System32\drivers\etc\hosts

```
127.0.0.1 www.abc.com
```

启动tomcat服务

启动ngnix服务

```
start ngnix.exe
```

注：start是windows的bat命令，表示启动一个exe命令。

4）访问服务

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

5）关闭

```
ngnix -s stop
```

注：-s表示signal

注2：帮助信息：nginx -?

6）日志查看

在ngnix解压目录下存在logs目录，该目录下有两个文件access.log和error.log文件，如果访问出现错误时，可以查看error.log文件。