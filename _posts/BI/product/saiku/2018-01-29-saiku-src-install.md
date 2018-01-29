---
title: saiku源码安装（未完成）
tags: [BI]
---

1）saiku的安装

下载地址：https://github.com/OSBI/saiku

将项目fork到自己的本地，然后checkout最新的一个版本，运行测试。

2）下载代码并测试运行

```
# 下载fork分支到本地
git clone https://github.com/yutian1012/saiku.git
# 查看tag
git tag -l
# 切换版本tag-3.9
git checkout tag-3.9
```

编译项目

```
mvn clean compile -Dmaven.test.skip=true
```

3）license去除

参考：http://blog.csdn.net/rotkang/article/details/78914866