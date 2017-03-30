---
title: myeclipse导入git中的maven项目
tags: [git]
---

### 1. 安装git插件（egit）

方法一：

下载egit压缩包，之后通过dropins+link或者其他方式安装egit插件

方法二：

myeclipse中--help--eclipse marketplace--search egit--finish

方法三：

myeclipse中--myeclipse configuration center--software--安装

方法四：

myeclipse中--Help--install from site--search git--安装

### 2. 将本地项目创建成git仓库

1）新建一个项目（可以是一个已有的项目，未加入到git管理中）

2）将项目纳入到git中管理

右键这个项目--team--share project--git--勾选单选框--点击项目--create repository--finish

3）git仓库已经建好了，可以在本地进行版本控制了

注：完成上面的配置，项目位于本地git仓库中

4）同步到远程git仓库

右键项目--team--remote--push--填写相关信息--finish。

注：远程仓库需要事先创建好一个目录，用来容纳该项目。

### 3. 从git仓库中检出到myeclipse中

方式一：使用命令行工具（前提系统中安装了git for windows工具）

```
git clone https://github.com/frohoff/ysoserial.git
```

然后在myeclipse中导入下载的源码，import--existing Maven Projects

方式二：

import--projects from git--Clone URI（输入git远程仓库的项目路径）--选择项目分支--设置项目的存储路径（clone下来的项目存放位置），会下载项目到本地--import existing projects（报错），可取消掉。

注：这时已经将项目下载到本地了，只需要import到工作空间即可（使用方式一将代码导入到工作空间）。

