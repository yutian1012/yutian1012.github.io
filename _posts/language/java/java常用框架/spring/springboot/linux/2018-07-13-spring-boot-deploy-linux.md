---
title: spring boot项目部署到linux下
tags: [spring]
---

首先完成linux系统的jdk配置（1.8以上）

1）打包项目

修改application.yml文件，完成数据库等相关环境的配置（可创建多份配置文件）

```
mvn clean package -Dmaven.test.skip
```

2）执行相关的建表语句

```
create database if not exists sipop;

```

3）安装redis

4）上传jar包并运行

```
java -jar xxx.jar
```

注：这种方式运行的jar包是在当前窗口运行的，当前窗口关闭了，程序自动退出。

后台运行

```
java -jar xxx.jar > /dev/null 2>&1 &
```

5）新建启动脚本

参考：https://blog.csdn.net/whh18254122507/article/details/78011713

```
touch startup.sh
# 分配权限
chmod +x startup.sh
```

编辑文档内容，或者线下编辑后上传

```
#!/bin/sh
RESOURCE_NAME=spider.jar

tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

case "$1" in
      start)
    if [ ${tpid} ]; then
        echo 'alread start!'
    else
        echo "Starting ..."  
        nohup java -jar ./$RESOURCE_NAME >/dev/null 2>&1 &
        echo $! > tpid
        echo 'start success!'
    fi
    ;;
  stop)
    if [ ${tpid} ]; then
        echo "Stopping ..."  
        kill -9 $tpid
        echo 'stoped success!'
    else
        echo 'not start!'
    fi
    ;;

  restart)
    if [ ${tpid} ]; then
        echo "Stopping ..."  
        kill -9 $tpid
        sleep 2;
    fi
    echo "Starting ..."  
    nohup java -jar ./$RESOURCE_NAME >/dev/null 2>&1 &
    echo $! > tpid
    echo 'restart success!'
    ;;
    
  *)
    echo "Usage $0 {start|stop|restart}"  
    ;;
esac
```