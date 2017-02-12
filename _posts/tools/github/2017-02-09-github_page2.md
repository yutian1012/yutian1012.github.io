---
title: github-page2
tags: [github,tools]
---

设置git ssh免密码登录，并将本地博客上传到github仓库。

### 1. 安装git（git for windows）
安装到E:\work\Git目录下

### 2. 设置Git的user name和email
打开命令行E:\work\Git\git-bash.exe，运行代码：

```
git config --global user.name "yutian1012"
git config --global user.email "yutian1012@126.com"
```

注：--global选项是针对所有用户都起作用的，会在~/.gitconfig文件中写入信息，windows系统即在C:\Users\xxx\目录下。

### 3. 查看ssh密钥
查看系统中是否存在ssh密钥

```
ll ~/.ssh
```

![](/images/tools/github/ssh.png)

注：windows系统C:\Users\xxx\.ssh目录下

如果有先删除，这里我们重新生成（删除前最好备份）

```
rm -r ~/.ssh
```

### 4. 生成密钥：
```
ssh-keygen -t rsa -C "yutian1012@126.com"
```

![](/images/tools/github/gensshkey.png)

运行后什么都不输入，直接回车即可。

注：-t指产生key的类型，-C指Comment，产生的公钥为id_rsa.pub，私钥为id_rsa

### 5. 在github上添加一个ssh密钥
拷贝C:\Users\elodie\.ssh\id_rsa.pub文件内容：登录github--个人信息--settings--SSH and GPG keys，用记事本打开id_rsa.pub，将内容粘贴到ssh的key文本框中。

![](/images/tools/github/githubaddssh.png)

### 6. 测试
测试代码：

```
ssh -T git@github.com
```

注：最后一行显示Hi yutian1012! You've successfully authenticated, but GitHub does not provide shell access.表示成功授权了。

### 7. 在github上创建一个github page仓库
github page要求仓库名必须为你github的名字+github+io（也就是我们以后访问的域名），即yourname.github.io。该仓库下将放置所有的站点博客信息。

### 8. 将本地博客系统上传到github -page上
注：将仓库clone到本地，在本地编辑博客内容，然后提交的github上显示。

1）在E:\work\blog目录下创建一个website目录，该目录将用来编辑站点。

打开cmd窗口，cd E:\work\blog\website

2）克隆github page项目到本地该目录下

```
git clone https://github.com/yutian1012/yutian1012.github.io.git
```

![](/images/tools/github/githubclone.png)

注：会在website目录下出现yutian1012.github.io文件夹，该文件夹就是从github上clone下的仓库。

3）建立起jekyll相关目录，并将博客内容放在到_posts下（参考本地博客系统搭建）

4）将文件添加到github上

```
git add .
git commit -m "my new blog"
git push origin master
```

注：git push <远程主机名> <本地分支名>:<远程分支名>，origin是默认的远程版本库名称。

### 9. 在浏览器中访问github
输入https://yourname.github.io即可访问博客，如：https://yutian1012.github.io/


资料（完整搭建）：https://bigballon.github.io/posts/jekyll-github.html
