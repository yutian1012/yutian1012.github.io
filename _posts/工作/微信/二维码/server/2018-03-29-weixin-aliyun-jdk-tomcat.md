---
title: 微信公共开发平台--阿里云服务器JDK
tags: [weixin]
---

参考：https://yq.aliyun.com/articles/503173?spm=a2c4e.11155472.0.0.10a26d15XE3kf8

1）安装jdk

下载jdk-8u161-linux-x64.rpm安装包，然后上传到阿里云服务器上。直接使用wget方式提示授权错误。

a.rpm方式安装jdk

```
# 查看版本信息
cat /proc/version
uname -a

# 查看是否安装了jdk
rpm -qa jdk
# 下载jdk安装包，wget方式不能正确安装，这里手动下载然后上传

# 安装
rpm -ivh jdk-8u161-linux-x64.rpm
```

b.设置环境变量

```
vi /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_151
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
```

c.使用yum方式安装

```
yum -y list java*
yum -y install java-1.8.0-openjdk.x86_64
```

注：这种方式安装的jdk不需要设置环境变量

c.运行java测试

```
java -version
```

2）安装tomcat

a.安装tomcat

```
cd /usr/local/src
wget -c http://mirrors.shuosc.org/apache/tomcat/tomcat-7/v7.0.85/bin/apache-tomcat-7.0.85.tar.gz
tar -xzvf apache-tomcat-7.0.85.tar.gz
mkdir /usr/local/tomcat
mv apache-tomcat-7.0.85 /usr/local/tomcat/
cd /usr/local/tomcat/apache-tomcat-7.0.85/
```

b.配置环境变量

```
vi /etc/profile
export TOMCAT_HOME=/usr/local/tomcat/apache-tomcat-7.0.85/
source /etc/profile
```

c.启动tomcat

```
cd /usr/local/tomcat/apache-tomcat-7.0.85/bin
./catalina.sh start
# 查看是否启动
curl -I localhost:8080
# 停止tomcat
./shutdown.sh
```

d.配置编码

```
cd /usr/local/tomcat/apache-tomcat-7.0.85
vi conf/server.xml
# 找到Connector port 8080处添加
URIEncoding="UTF-8"
```