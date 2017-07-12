---
title: git安装（windows）
tags: [git,tools]
---

windows安装msysgit（Windows版的Git）

1）下载

下载地址https://git-for-windows.github.io，下载完成后直接按照默认选项安装即可。

安装完成后在cmd中输入命令查看

```
git --version
```

注：安装完成后会自动设置环境变量

2）配置github的

在cmd中运行命令

```
git config --global user.name "yutian1012"
git config --global user.email "yutian1012@126.com"
```

注：--global选项是针对所有用户都起作用的，会在~/.gitconfig文件中写入信息，windows系统即在C:\Users\xxx\目录下。

3）使用git clone命令下载代码