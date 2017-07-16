---
title: 检出tag下的项目
tags: [git]
---

参考：http://yijiebuyi.com/blog/eacf4d053fad77affffae397d9af7172.html

场景：在使用actitivi时，目前项目开发使用的是activiti.5.10，不希望检出最新的代码。而只是坚持tag下的5.10源码。

1）下载项目

首先本地没有下载activiti项目的git项目，要先下载

```
git clone https://github.com/Activiti/Activiti.git
```

注：这里检出的是master分支的最新版本

2）检出tag下的5.10版本

运行命令，查看tag

```
git tag
```

checkout下5.10版本

```
git checkout activiti-5.10
```

注：原来的master代码会被5.10版本的代码覆盖掉。

3）总结

不能在没有任何版本库信息的情况下直接checkout相应的tag。另外，git切换分支也是类似的。