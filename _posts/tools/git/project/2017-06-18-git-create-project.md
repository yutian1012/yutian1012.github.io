---
title: git创建项目并使用eclipse开发
tags: [git]
---

实现先在eclipse中创建项目，然后上传到github上。

### eclipse中项目上传到github上

参考：http://www.mamicode.com/info-detail-928508.html

1）首先在github网站创建远程仓库

仓库地址为：https://github.com/yutian1012/reflect.git

2）创建本地仓库

在eclipse上创建项目后，右击项目--team--share Project

![](/images/tools/git/project/share_project.png)

选择项目所在文件夹作为git的本地仓库

![](/images/tools/git/project/share_project_local_repository.png)

![](/images/tools/git/project/share_project_create_local_repository.png)

会在项目中出现.git文件夹

![](/images/tools/git/project/project_git_flod.png)

3）编辑忽略提交的文件

打开.gitignore文件，编辑内容

```
/target
/.settings
```

4）上传代码到远程

提交代码到本地仓库

![](/images/tools/git/project/project_commit_local.png)

![](/images/tools/git/project/project_commit_local_code.png)

将本地仓库内容同步到github仓库上

![](/images/tools/git/project/project_remote_set.png)

![](/images/tools/git/project/project_remote_set2.png)

![](/images/tools/git/project/project_remote_set3.png)

注：Add All Branches Spec按钮点击后会变成灰色的（不可再点）

5）查看github上的仓库

![](/images/tools/git/project/project_remote_set4.png)

注：发现文档已经提交到了github远程仓库上了