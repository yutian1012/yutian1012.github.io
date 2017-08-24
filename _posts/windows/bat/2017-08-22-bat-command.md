---
title: bat中常用命令
tags: [bat]
---

1）清空屏幕

```
cls
```

2）空格问题

文件路径可以有空格，命令参数也可以有空格，但命令路径执行时不能有空格。

命令参数中有空格，能够正常执行

```
cd >notepad C:\Program Files\Common Files
notepad C:\Program Files\Common Files\sss.txt
```

```
D:\Program Files\Java\jdk1.7.0_75\bin\java
```

注：将D:\Program看作是命令，空格分隔的为参数，因此报错。

3）mkdir命令

错误命令

```
mkdir a/b/c
错误信息--命令语法不正确
```

正确命令：

```
mkdir a\b\c
```

注：会在当前目录下创建a\b\c嵌套目录。