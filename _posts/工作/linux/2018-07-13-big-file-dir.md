---
title: 大文件的查找与删除
tags: [linux]
---

查找大的文件夹

```
# 从根目录下开始查找，指定进一步确定大文件的位置
du -h --max-depth=1 /
```

发现/var/spool/clientmqueue目录下存在着大量的文件，占用了磁盘空间的80%

1）/var/spool/clientmqueue目录的作用

系统中有用户开启了cron，而cron中执行的程序有输出内容，输出内容会以邮件形式发给cron的用户，而sendmail没有启动所以就产生了这些文件。

2）解决方法

```
将crontab里面的命令后面加上> /dev/null 2>&1
```

2>&1：把错误重定向到输出要送到的地方，即重定向到/dev/null