---
title: SVN（Subversion）提交注释
tags: [svn]
---

在svn提交代码后，想对注释再次修改。

参考：https://zhidao.baidu.com/question/323568989.html

1）添加pre-revprop-change.bat

windows系统下，在该项目的仓库下面找到hooks目录，在下面新建一个空文件。pre-revprop-change.bat。注意，后缀名是bat。

首先定位到svn存储项目的目录下

![](/images/tools/svn/log/svnhooks.png)

2）修改log备注

在eclipse的svn插件上未找到修改备注的功能，这里可以使用tortoiseSVN工具来实现。首先定位到修改的版本号，找到待修改的svn提交版本，右击选择 edit log message.

![](/images/tools/svn/log/tortoiseSVNshowlog.png)

![](/images/tools/svn/log/tortoiseSVNeditlogmessage.png)