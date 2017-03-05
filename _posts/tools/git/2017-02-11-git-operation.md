---
title: git常用操作
tags: [git,tools]
---

### 1. git中origin的含义是什么？

origin只相当于一个别名，运行git remote –v或者查看.git/config可以看到origin的含义。

```
git remote -v
```

注：该命令可以看出origin其实就是远程的git地址的一个别名。

### 2. 分支操作：
参考：http://blog.csdn.net/arkblue/article/details/9568249/

1）查看远程分支：

```
git branch -a
```

注：会同时显示本地分支和远程分支

2）查看本地分支：

```
git branch
```

3）创建分支

```
git branch develop
```

注：创建本地分支develop，创建前先提交本地库，否则分支信息中不包含未提交的操作。

4）把分支提交到远程服务器
```
git push origin develop
```

5）删除本地分支

```
git branch -d develop
```

6）删除远程分支

```
git branch -r -d origin/develop
或
git push branch :develop
```

注：第二种方式相当于将一个空的分支推送到远程。

### 3. 提交操作

将当前目录下的文件add到本地仓库

```
git add .
```

注：后面为提交的目录，会变量该目录下的所有文件和文件夹，提交修改。

查看当前目录下的文件状态

```
git stauts .
```

提交到本地仓库

```
git commint . -m 'modify'
```

