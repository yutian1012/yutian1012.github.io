---
title: startUml根据源码生成类图
tags: [activiti]
---

为了更方便的查看activiti下的源码结构，可以通过StartUml工具的逆向工程，生成代码的类图。

参考：http://blog.csdn.net/qq_20480611/article/details/51287038

1）创建project

在右侧Model Explorer中修改uml项目名称，并保存

![](/images/book/workflow/activiti/uml/uml-project.png)

2）使用插件进行逆向工程

添加相关语言的插件才能逆向出类图。点击菜单栏【Tools】—【Add-In-Manager】,然后勾选中【Java Add-In】,然后点击【OK】就完成了对应插件的安装（默认已经全装上了）。

![](/images/book/workflow/activiti/uml/uml-plugin.png)

3）生成java的逆向图前添加profile

profile为生成java逆向工程的一个环境，在执行逆向工程前需要把环境配好。

点击菜单栏【Model】—【Profile】，将java Profile添加到右侧inculded Profiles.

![](/images/book/workflow/activiti/uml/uml-profile.png)

4）执行逆向工程

在右侧uml项目上右击--java--ReverseEngineer..，或者在菜单中【Tools】-【java】-【Reverse Engineer...】来执行逆向工程。

![](/images/book/workflow/activiti/uml-revese-engineer.png)

添加源文件：

执行逆向工程时，需要选择相关的java源代码所在的文件夹（最外层代码）。这样会将文件夹内的所有java代码都添加进来。

![](/images/book/workflow/activiti/uml-revese-engineer-addfiles.png)

类图存放位置：

选择存放生成图的文件夹（这里选择Design Model，设计模型）

![](/images/book/workflow/activiti/uml-revese-engineer-package.png)

注：由于staruml只支持jdk1.3的，所以当代码中有泛型或者注解 for  in等高级特性时，生成类图会失败