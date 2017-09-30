---
title: git配置（windows）
tags: [git,tools]
---

参考：http://blog.csdn.net/zxncvb/article/details/22153019




### git访问远程仓库方式

参考：https://help.github.com/articles/changing-a-remote-s-url/

1）使用命令

```
git remote set-url
```

2）访问方式（https和ssh方式）

```
//use HTTPS
https://github.com/USERNAME/REPOSITORY.git

// use SSH
git@github.com:USERNAME/REPOSITORY.git
```

3）查看设置

```
git remote -v
```

4）设置

```
// set https
git remote set-url origin https://github.com/USERNAME/REPOSITORY.git

//set SSH
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
```