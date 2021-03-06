---
title: 开源deploy code自动化部署工具（开发思路）
tags: [deploy]
---

### 问题：

针对当前项目代码更新部署，部署工作比较麻烦，而且经常会出现忘记更新某个文件或数据库字段忘记添加字段，导致部署工作进程会出现问题。

### 思路：

借助svn，git等版本控制管理工具实现自动部署。

### 环境：

1）项目分支：主干master，预发布分支pre-deploy（主干上的分支），开发分支（在预处理分支上创建的分支）。
2）开发部署环境：开发环境（开发人员本地机器），预发布环境（类似线上环境），线上环境。

注：预发布环境是和线上环境互通的（能相互ping通，并能拷贝文件）

### 实施：

1）开发代码并提交

本地人员先在本地修改代码，代码修改完成后将其提交到开发分支上。等到所有人都开发完成后，统一提交后。将开发分支合并到预发布分支上。

2）从预发布分支上获取更新代码

可以是版本控制工具提供的hook，获取合并时更新的代码。

3）swing管理部署工具/web类型的部署工具

可以开发一个swing的管理工具，管理工具的主要功能是定义部署的项目路径，通过内置的ant编译环境编译出更新包/全部软件包，并提供导出和下载功能。同时也可以提供相应的源码包，包含更新的源代码和全部软件的源代码。

4）在预发布服务器上测试

线上预发布方式可以分成两种：

方式一：使用更新包（编译后的class），需要手动的执行sql并替换文件，然后测试运行（传统方式）

方式二：swing管理部署工具/web类型的部署工具（自开发）。可以上传更新包/全软件包，也可以结合预发布服务器安装的svn等版本工具实现向指定地址发布软件包，并实现自动执行sql，自动覆盖。

5）将更新发布到线上服务器

发布方式与预发布类似。预发布测试完成后，可以通过部署工具一键发布到线上（预发布服务器和线上服务器互通），也可通过手动执行拷贝方式实现。

###场景：

一般来讲，线上环境与内外环境是分离的，不能直接访问，必须通过一个堡垒机登录后，才能访问线上环境。因此现在的做法是将待更新的代码制作成压缩包，登录堡垒机后，将压缩包拷贝到线上环境中，重启服务。

在预发布服务器上创建一个svn，将源码包覆盖指定的工作空间，然后提交，通过提交可获取改动的项目内容，进一步导出部署代码与开发环境下的部署导出类似。

注：svn会部署到开发环境所致的内网和预发布服务器所在的环境，部署管理工具则可以安装到相应的svn服务器所在的机器上。