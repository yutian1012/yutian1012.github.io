---
title: git 回退操作
tags: [architecture]
---

1）回退到某个版本

```
# 查看版本号信息
git log

# 指定具体版本号
git reset --hard 1094a
```

2）放弃本地修改

```
git checkout .
```

注：并没有提交

3）放弃回退版本

```
git pull
```