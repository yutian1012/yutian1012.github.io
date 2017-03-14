---
title: git操作实践
tags: [git,tools]
---

### 1. git add -p 命令

以"块"形式暂存改动，add命令会将信息存储到暂存区（staging area）。主要实现对文件的修改实现部分提交。

场景：你可能也会偶然因为两个不同的原因而做了一次改动，却没有分别提交（仅仅提交了一次），所以，当你执行git log时，会看到诸如这样的提交信息："修改X，改动无关的Y"。

1）添加一个文件test.txt，并提交到本地仓库。

```
git add test.txt
git commint test.txt -m 'addtestfile'
```

2）在test文档中编辑三行

```
helloworld
test
welcome
```

并使用git diff比较当前文档与暂存文件的不同

```
git diff test.txt
```

输出内容：

```
diff --git a/test.txt b/test.txt
index e69de29..b83d40a 100644
--- a/test.txt
+++ b/test.txt
@@ -0,0 +1,3 @@
+helloworld
+test
+welcome
\ No newline at end of file
```

注：可以看到test.txt文档增加了3行.

add文档到暂存中

```
git add test.txt
```

3）编辑文档，在文档开头添加并在文档末尾添加文字

```
add first modify
helloworld
test
welcome
add last modify
```

注：修改的内容被分割在不同位置。

git diff命令查看

```
diff --git a/test.txt b/test.txt
index b83d40a..71b85b2 100644
--- a/test.txt
+++ b/test.txt
@@ -1,3 +1,5 @@
+add first modify
 helloworld
 test
-welcome
\ No newline at end of file
+welcome
+add last modify
\ No newline at end of file
```

注：git可以根据不同的修改位置对修改进行分块处理。

4）使用git add -p命令分块add到暂存中

具体操作，只提交最后的修改，吧第一行的修改忽略掉。

```
git add -p test.txt
```

![](/images/tools/git/gitadd-p.png)

命令：输入y来缓存该块，输入n不缓存该块，输入e来人工编辑该块，输入d来退出或进入下一个文件，输入s来分割这个块。

注：可以看出第一块是源文件的1-2行发生了变化，即在开头增加了一行，输入n表示不add该块。第二块是文件末尾的修改，输入y表示add该块到暂存区。

注2：源文件想象成改动前的文件，该文件暂存在暂存区中。

操作完成后，使用git dff查看

```
diff --git a/test.txt b/test.txt
index cb03385..71b85b2 100644
--- a/test.txt
+++ b/test.txt
@@ -1,3 +1,4 @@
+add first modify
 helloworld
 test
 welcome
```

注：可以看到本地工作区和暂存区的区别体现在本地工作区在第一添加了数据，而暂存区没有该数据。

5）如何查看暂存区中的内容

可以通过checkout方式将暂存区内容覆盖本地工作区内容，从而看到暂存区内容。如果覆盖了，那么在第一行添加到数据就找不到了。

运行git status查看当前状态

```
D:\work\installer\git_home\webmodel Maven Webapp>git status
On branch master
Changes to be committed:
  --提示放弃暂存区的修改，用本地仓库内容覆盖暂存区，本地工作区不受影响
  (use "git reset HEAD <file>..." to unstage)

        modified:   test.txt

Changes not staged for commit:
  --提示将本地区add到暂存区
  (use "git add <file>..." to update what will be committed)
  --提示放弃本地区修改，使用暂存区覆盖本地区，从而可以看到暂存区内容
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   test.txt
```

注：可以看到test.txt文件修改没有被提交，并且暂存区与本地工作区也是不同的

```
git checkout -- test.txt
```

执行完成命令后，打开文件发现未提交的第一行数据的修改没有了。

文档内容：

```
helloworld
test
welcome
add last modify
```

### 2. git reset命令
参考：http://www.cnblogs.com/kidsitcn/p/4513297.html

git reset命令是用来将当前branch重置到另外一个commit的

1）代码修改后，add到暂存区，没有commit到本地仓库

```
git reset HEAD filepath
```

注：暂存区会与本地仓库最近commit的值相同，不会影响本地工作区内容

2）代码修改后，add到暂存区，同时commit到本地仓库（实现回滚提交）

```
git reset --soft HEAD^
```

错误提示：fatal: ambiguous argument 'HEAD

注：^在cmd中是特殊的字符，需要使用双引号括起来

```
git reset --soft HEAD"^"
或者
git reset --soft HEAD~1
```

运行git diff --cached filepath命令，可以看到该文件的提交被回滚了，暂存区内容不变。

3）添加文件，add到暂存区，恢复到添加文件前

```
git reset --hard
相当于
git reset --hard HEAD
```

注：会恢复到当前commit版本，暂存区，工作目录区都会退回到上一个commit状态。

注：--soft会影响暂存区，--hard会影响暂存区和工作目录区。

### 3. git命令补全

适用于linux环境下。

参考：http://www.cnblogs.com/memory4young/p/git-command-auto-completion.html

1）下载 Git 的源代码

```
git clone https://github.com/git/git
```

2）复制 git-completion.bash

cd git/contrib/completion/

注：源代码下有个contrib/completion目录，该目录下有git-completion.bash文件

将该文件复制到主目录(~)下

```
cp git-completion.bash ~/.git-completion.bash
```

注意：复制时，文件名前加一个"点"（.）

3）修改主目录下的 .bashrc 文件

```
vi ~/.bashrc
```
 
在文件的最后一行，加上如下代码：

```
source ~/.git-completion.bash
```

4）测试命令补全功能

```
git co<tab><tab>
```

注：会得到git commit命令。

### 4. 查看文件中的修改信息

场景：用于查找每行代码的修改人员，便于纠错和追责。

```
git blame [file_name]
```

注：这个命令会显示文件中每一行的作者，每次改动后进行的提交(commitID)以及提交的时间戳。blame是纠错，指责的意思，即找出这个文件都是谁改的，从而确定出错误的提交人。

### 5. git log命令

参考：http://blog.csdn.net/wh_19910525/article/details/7468549

1）查看提交历史

```
git log
```

注：默认不用任何参数的话，git log会按提交时间列出所有的更新，最近的更新排在最上面。

2）日志中查看提交差异

```
git log -p -2
```

注：-p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新

场景：在做代码审查，或者要快速浏览其他协作者提交的更新都作了哪些改动时，就可以用这个选项。

3）查看日志的统计信息

```
git log --stat
git log --stat test.txt 单独查看一个文件的修改提交统计情况
```

注：每个提交都列出了修改过的文件，以及其中添加和移除的行数，并在最后列出所有增减行数小计。

注意：退出时按q键。

4）日志的pretty选项

```
git log --pretty=oneline test.txt 单独查看一个文件
```

注：oneline将每个提交放在一行显示，只显示commit ID和备注信息。

```
git log --pretty=format:"%h - %an, %ar : %s" test.txt
```

注：使用格式化结果显示日志信息。

5）查看最近的提交日志

```
git log --since=2.weeks test.txt
```

注：最近两个星期的提交

```
-(n) 仅显示最近的 n 条提交
--since, --after 仅显示指定时间之后的提交。从什么时候开始的提交日志
--until, --before 仅显示指定时间之前的提交。到什么时候结束的提交日志
--author 仅显示指定作者相关的提交。
--committer 仅显示指定提交者相关的提交
```

6）使用图像化的界面查看日志

```
gitk
```

注：相当于 git log 命令的可视化版本，凡是git log 可以用的选项也都能用在 gitk 上。在项目工作目录中输入 gitk 命令后，就会启动日志查看界面。

![](/images/tools/git/gitk.png)

参考：http://blog.jobbole.com/75348/