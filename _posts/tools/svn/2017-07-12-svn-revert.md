---
title: SVN（Subversion）退回操作
tags: [svn]
---

SVN回退到上一个版本

1）未提交退回修改

在本地修改了一些代码，却不想提交，可以使用revert回滚到未修改时的版本。

在文件上右击--team--revert。

![](/images/tools/svn/revert/localrevert.png)

2）已修改回滚（update to，无法提交）

在本地修改了代码，并提交到了svn，现想回滚到上一个版本。

方式一：文件上右击--team--show history，在相应回滚到的版本上右击--update to

![](/images/tools/svn/revert/updateto.png)

方式二：文件上右击--team--update to Revision，然后选择相应的回滚版本。

注：使用update to方式不能提交修改

执行team--update即可恢复到最新版本

3）已修改回滚（get content）

在本地修改了代码，并提交到了svn，现想回滚到上一个版本。

文件上右击--team--show history，在相应回滚到的版本上右击--get content

![](/images/tools/svn/revert/getcontent.png)

再次提交会产生一个新的版本号