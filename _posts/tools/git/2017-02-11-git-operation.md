---
title: git常用操作
tags: [git,tools]
---

### 1. git中的区域概念

git区域包括：供作区，暂存区，仓库区（或本地仓库），远程仓库

![](/images/tools/git/gitzone.png)

```
Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库
```

注：不同命令操作所改变的状态和区域是不同的。


### 2. git创建项目（实例）

1）定位到eclipse创建的项目目录下，执行git初始化操作

```
cd D:\work\installer\git_home\webmodel Maven Webapp
git init
```

注：执行git init后会在该目录下创建.git文件夹。该文件夹Git 仓库，项目的快照数据都存放在这里。

2）创建.gitignore文件，存储忽略的文件信息（配置不需要加入版本管理的文件）

![](/images/tools/git/gitignore.png)

一般项目中的.settings、.classpath、.myumldata、.project、.springBeans、target文件是没有必要上传到服务器上的。

编辑.gitignore文件

```
.settings/
target/
.classpath
.myumldata
.project
.springBeans
```

注：一行一个文件或目录名，也可以使用通配符来匹配文件。

可以通过git status命令查看未忽略的文件

```
git status
```

![](/images/tools/git/gitnotignorefiles.png)

3）将本地仓库上传到服务器（这里服务器就使用github）

注：要上传到github仓库，就需要先在github上有一个对应的仓库位置。

第一步：首先在github上创建仓库，仓库名为：webmodel

![](/images/tools/git/githubrepository.png)

创建后的仓库地址为：

```
https://github.com/yutian1012/webmodel.git
```

![](/images/tools/git/githubpushrepository.png)

注：github上的空项目中也给出了如何上传项目到github上的操作。

第二步：其次，本地仓库与远程仓库关联

```
git remote add origin https://github.com/yutian1012/webmodel.git
git remote -v
```

第三步：在push到远程仓库前需要先在提交commit到本地仓库

```
git add .
git commit . -m 'init commit'
```

在执行git commit时会出现错误，需要先配置身份

```
*** Please tell me who you are.
fatal: empty ident name (for <(null)>) not allowed
```

第四步：配置身份

```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

注：运行命令后会在用户的根目录下生存一个.gitconfig文件，文件内容为用户配置的信息。

配置完成后，就可以正确的执行commit命令了。

第五步：将本地仓库内容提交到服务器，需要输入用户名和密码

```
git push -u origin master
```

注：到github上查看，就可以看到新建的项目内容了

### 3. git ssh生成密钥

利用git安装目录下（Git\usr\bin）的工具ssh-keygen.exe，生成出一个公钥和私钥对。

```
ssh-keygen -t rsa
```

![](/images/tools/git/sshkeygen.png)

复制id_rsa.pub文件内容到github上，设置SSH and GPG keys，添加一个SSH key。

测试

```
ssh -T git@github.com
```

### 4. git免用户名和密码输入

虽然配置了ssh的key，但git仓库的远程使用的仍然是https方法，这种方式就需要用户手动输入用户名和密码，可以该为ssh方式。

查看remote方式

```
git remote -v
//输出内容
origin  https://github.com/yutian1012/webmodel.git (fetch)
origin  https://github.com/yutian1012/webmodel.git (push)
```

修改ssh方式

```
git remote rm origin
git remote add origin git@github.com:yutian1012/webmodel.git
```

注：'git@github.com'是固定字符串，'yutian1012'是git用户名，'webmodel.git'是仓库

再次执行push命令就不会再提示输入用户名和密码了

```
git push origin master
```

### 5. 分支操作：

参考：http://blog.csdn.net/arkblue/article/details/9568249/

1）查看远程分支：

```
git branch -a
```

注：会同时显示本地分支和远程分支

2）查看本地分支：

```
git branch
```

3）创建分支

```
git branch develop
```

注：创建本地分支develop，创建前先提交本地库，否则分支信息中不包含未提交的操作。

4）切换分支

```
git checkout develop
```

![](/images/tools/git/gitbranchcheck.png)

注：星号表示工作空间所在的分支。

5）把分支提交到远程服务器
```
git push origin develop
```

6）删除本地分支

```
git branch -d develop
```

7）删除远程分支

```
git branch -r -d origin/develop
或
git push branch :develop
```

注：第二种方式相当于将一个空的分支推送到远程。

### 6. 提交操作

将当前目录下的文件add到本地仓库

```
git add .
```

注：后面为提交的目录，会变量该目录下的所有文件和文件夹，提交修改。

查看当前目录下的文件状态

```
git stauts .
```

提交到本地仓库

```
git commint . -m 'modify'
```

推送到远程仓库

```
git push origin master
或
git push origin master:master
```

注：提交到远程的master分支上

### 7. git diff命令

1）git diff比较的是工作目录与暂存区的区别

在一个test.txt文件中编辑数据，编辑内容未提交到暂存区（Stage），使用git diff查看

向空文件test.txt文件中添加内容

```
first modified

second modified

third modified
```

查看

```
git diff test.txt
```

![](/images/tools/git/gitdiff.png)

2）git diff --cached比较的是暂存区与本地仓库的区别

在add到暂存区前先查看git diff --cached输出

```
git diff --cached test.txt
```

注：输出空，即暂存区和本地仓库没有变化（两种文件相同）

将文件信息提交到暂存区

```
git add test.txt
```

再次运行git diff命令

```
git diff test.txt
```

注：输出空，从而也能够得出git diff命令查看的是本地工作区与暂存区的文件区别这一结论，因为add操作后现在两者的文件内容相同了，所以输出空。

执行git diff --cached命令，看是否还是输出空

```
git diff --cached test.txt
```

![](/images/tools/gti/gitdiffcached.png)

输出内容：

```
diff --git a/test.txt b/test.txt
index e69de29..8831aa6 100644
--- a/test.txt
+++ b/test.txt
@@ -0,0 +1,5 @@
+first modified
+
+second modified
+
+third modified
\ No newline at end of file
```

注：这里的a/test.txt表示的是本地仓库文件，b/test.txt表示暂存空间文件

问题：会不会是本地目录文件与仓库文件的区别？如何排除

在本地空间中向test.txt中追加数据。

运行命令：git diff test.txt

![](/images/tools/git/gitdiffstage.png)

输出内容：

```
D:\work\installer\git_home\webmodel Maven Webapp>git diff test.txt
diff --git a/test.txt b/test.txt
index 8831aa6..78f70e5 100644
--- a/test.txt
+++ b/test.txt
@@ -2,4 +2,6 @@ first modified

 second modified

-third modified
\ No newline at end of file
+third modified
+
+fourth modified
\ No newline at end of file
```

证明：git diff比较的是工作目录与暂存区的区别

运行命令：git diff --cached test.txt

![](/images/tools/git/gitdiffcached.png)

输出内容与上面输出相同：

```
diff --git a/test.txt b/test.txt
index e69de29..8831aa6 100644
--- a/test.txt
+++ b/test.txt
@@ -0,0 +1,5 @@
+first modified
+
+second modified
+
+third modified
\ No newline at end of file
```

证明：--cached参数表示的是暂存区与本地仓库的区别

3）git diff HEAD 比较的是工作目录与本地仓库的区别

运行命令：

```
git diff HEAD test.txt
```

![](/images/tools/git/gtidifflocal.png)

输出内容：

```
diff --git a/test.txt b/test.txt
index e69de29..78f70e5 100644
--- a/test.txt
+++ b/test.txt
@@ -0,0 +1,7 @@
+first modified
+
+second modified
+
+third modified
+
+fourth modified
\ No newline at end of file
```

4）git输出信息分析

```
diff --git a/test.txt b/test.txt
index e69de29..8831aa6 100644
--- a/test.txt
+++ b/test.txt
@@ -0,0 +1,5 @@
+first modified
+
+second modified
+
+third modified
\ No newline at end of file
```

注：这里的a/test.txt表示的是暂存空间文件，b/test.txt表示本地工作空间文件

第一行：diff --git a/test.txt b/test.txt

进行比较的是，a版本的test.txt（即变动前，暂存区）和b版本的test.txt（即变动后，本地工作目录区域）

第二行：index e69de29..8831aa6 100644

表示两个版本的git哈希值（index暂存区域的e69de29对象，与工作目录区域的8831aa6对象进行比较），最后的六位数字是对象的模式（普通文件，644权限）。  

第三、四行：--- a/test.txt +++ b/test.txt

表示进行比较的两个文件，"---"表示变动前的版本，"+++"表示变动后的版本。

注：后面行是git diff的比较结果

第五行：@@ -0,0 +1,5 @@

这里表示变动前文件的0行开始0行结束处，与变动后文件的第1行开始和第5行结束处的比较内容结果（即我们向空文件test.txt中添加了5行内容）。

注：差异按照差异小结进行组织，每个差异小结的第一行都是定位语句，由@@开头，@@结尾

比较结果输出：

```
+first modified
+
+second modified
+
+third modified
\ No newline at end of file
```

注：输出的前缀主要分为-，+和空格，含义如下。

```
-开头的行，是只出现在变动前文件中的行
+开头的行，是只出现在变动后文件中的行
空格开头的行，是变动前文件和变动后文件中都出现的行
```



### 7. git常用命令(阮一峰）

参考：http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html

1）新建代码库

```
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

2）配置

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件--会启动一个vi编辑器
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

3）增加/删除文件

```
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录和目录下的文件
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区，
# 前提是文件已经被add了，否则直接删除即可
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区，并没有提交，需要单独提交
$ git mv [file-original] [file-renamed]
```

注：rm和mv后可以使用git status查看当前文件的状态。

4）代码提交

```
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 可实现重命名提交备注信息
# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

5）分支

参考：http://blog.sina.com.cn/s/blog_63eb3eec0101fgex.html

```
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```