---
title: git常见错误
tags: [git,tools]
---

1）There is no tracking information for the current branch

执行git pull命令时，抛出异常信息

```
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
```

执行命令：

```
git branch --set-upstream-to=origin/master master
```

或者每次执行时都需要完整的分支

```
git pull origin master
```

注：origin指向的是repository，master只是这个repository中默认创建的第一个branch。