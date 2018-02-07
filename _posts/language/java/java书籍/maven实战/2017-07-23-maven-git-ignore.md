---
title: maven多模块下忽略target等目录的git上传
tags: [maven]
---

场景：maven是一个多模块的聚合项目，因此该项目下包含很多的子模块。现在想把该项目上传到git服务器上，要忽略到所有项目的target等编译目录。

![](/images/book/maven/scm/maven-git-ignore.png)

注：图片中可以看到target等文件上有问号，这说明该文件会被提交到git仓库中

解决方法：

在聚合项目下添加.ignore文件，并编辑文档内容

```
.classpath
.project
target
```

注：当前目录，子目录皆可忽略。