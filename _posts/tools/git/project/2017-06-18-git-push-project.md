---
title: git本地创建项目并上传到github上
tags: [git,tools]
---

实现将本地的项目上传到github上

###  git创建项目（实例）

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

注：运行命令后会在系统中用户的根目录下生存一个.gitconfig文件，文件内容为用户配置的信息。

配置完成后，就可以正确的执行commit命令了。

第五步：将本地仓库内容提交到服务器，需要输入用户名和密码

```
git push -u origin master
```

注：到github上查看，就可以看到新建的项目内容了