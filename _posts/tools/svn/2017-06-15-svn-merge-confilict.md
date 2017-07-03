---
title: SVN冲突合并
tags: [svn]
---

参考：http://www.cnblogs.com/jose-Lee/p/4896088.html

### 冲突的形式

参考：http://www.cnblogs.com/chengxiaohui/articles/2705073.html

冲突的分隔符：以<<<<<<<、=======、>>>>>>>分隔。

<<<<<<< 和=======之间是本地修改的内容（自己修改的），=======与>>>>>>>之间是远程获取的版本修改的内容（别人修改的）。

### 合并分支错误文件

如果在合并分支时出现错误，一般会出现4个文件

xxx.working 为合并前本地版本内容（本地版本）

xxx.merge-left.r.664：为664版本，即创建分支时主干中的版本（最初的版本）

xxx.merge-right.r665：为665版本，即合并前remote对应的svn分支中的版本

xxx 合并产生冲突的文件


### 主要有三种方式，

1）revert方式

修改冲突文件，修改完成后，复制文件内容，revert文件，然后粘贴回去。

2）resolve方式

需要利用tortoiseSVN工具，解决完成冲突后，右键修改完成的文件，选择tortoiseSVN-resolve。

注：这种方式在eclipse中的svn插件是不支持的。

3）eclipse中svn插件

在Team Synchronizing视图下可以解决冲突后，右击文件选择mark as resolve或者overide and update选项。

![](/images/tools/svn/merge/solveconflict.png)

更新修改内容

![](/images/tools/svn/merge/acceptchanges.png)