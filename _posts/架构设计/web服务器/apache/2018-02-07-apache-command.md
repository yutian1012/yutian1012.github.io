---
title: apache常用命令
tags: [web服务器]
---

### windows系统下

在使用cmd访问时，要使用administrator用户来运行，否则某些命令是无法正常执行的。

1）查看常用命令

```
httpd -h
```

2）检查配置文件是否正确

```
httpd -t
```

3）查看Apache安装了哪些编译模块

```
httpd -l
```

4）启动httpd服务

```
httpd -k start
```

5）关闭httpd服务

```
httpd -k stop
```

6）重启服务

```
httpd -k restart
```