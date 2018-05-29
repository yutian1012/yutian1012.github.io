---
title: SVN（Subversion）修改分支名称
tags: [svn]
---

1）在svn服务器上定位安装目录

进入到svn安装目录的bin目录下，运行如下命令

```
# 查看svn版本号
svn --version

# 查看仓库分支，使用文件路径来查看，也可以使用svn方式查看
svn list https://172.17.30.102/svn/eip-saas/branches

# 修改分支名称，使用move命令修改分支名
svn move -m '修改分支名称' https://172.17.30.102/svn/eip-saas/branches/eip-saas-whlg20180529 https://172.17.30.102/svn/eip-saas/branches/uip20180529
```

2）在eclipse中重新定位

右键点击项目--team--switch切换到新的svnurl地址上；在SVN-Repositories视图中断开被重命名的分支；右键点击项目--team--share project，重新建立关联。