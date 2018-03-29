---
title: git忽略文件
tags: [git]
---

参考：https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013758404317281e54b6f5375640abbb11e67be4cd49e0000

1）创建.gitignore文件

在git工作空间下创建.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。

```
# 忽略target文件夹下的内容
target/
# 忽略所有以.class结尾的文件
*.class
```

GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：https://github.com/github/gitignore

2）忽略文件的原则是：

a.忽略操作系统自动生成的文件，比如缩略图等；

b.忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件；

c.忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

3）eclipse项目下忽略的文件设置

```
target/
.settings/
.classpath
.project
.gitgnore
```

注：如果此时git远程仓库中已经存在相关文件，可手动进行删除。

4）删除远程文件和文件夹

```
git rm -r --cached .settings
git rm -r --cached target
git rm --cached .classpath
git rm --cached .project

git -f clean
git commit -m 'delete files'
git push -u origin master
```