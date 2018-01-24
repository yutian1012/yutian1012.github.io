---
title: git回退操作
tags: [git,tools]
---

参考：https://www.cnblogs.com/qufanblog/p/7606105.html

1）未使用git add缓存代码时

```
# 放弃单个文件的修改，不要忘记中间的--，否则就成了检出代码了
# 如git checkout -- readme.md
git checkout -- filepath_name
# 放弃所有的文件修改
git checkout .
```

注：放弃所有修改命令不会删除掉刚新建的文件。因为刚新建的文件还没已有加入到git的管理系统中，所以对于git是未知的。自己手动删除就好了。

2）已经使用了git add缓存的代码

清除git对于文件修改的缓存。相当于撤销git add命令所作的工作。在使用本命令后，本地的修改并不会消失，而是回到了未缓存的状态。继续上一步中的操作，就可以放弃本地的修改。

```
# 放弃指定文件的缓存
# 如git reset HEAD readme.md
git reset HEAD filepath_name
# 放弃所有的缓存
git reset Head .
```

3）已经使用git commit提交了代码

可以用来回退到任意版本。回退时结合log日志信息，确定回退的版本，然后进行回退。

```
# 回退到上一个版本
git reset --hard HEAD^
# 指定退回的版本号，版本号没必要写全，前几位就可以了，Git会自动去找
# 如git reset --hard 3628164
git reset --hard <commit-id>
```