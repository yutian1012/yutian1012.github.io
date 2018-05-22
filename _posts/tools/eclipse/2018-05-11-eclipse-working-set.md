---
title: eclipse中使用working set管理项目
tags: [eclipse]
---

参考：https://blog.csdn.net/yasi_xi/article/details/43269815

1）working set

在eclipse中可能会存在多个项目，而有些项目是相关联的，而有些项目却与其他项目又毫无联系。如果都在一个工作空间中，查找起来非常不方便，而且不方便组织及查看。

方案一：

创建多个工作空间，将不同的项目分别放在到不的工作空间中，查看使用相应的项目时，只需要切换到不同的工作空间中即可。

菜单命令：file--switch workspace

注：这种方式虽然实现了项目的分开治理，但是不方便来回切换，效率低。

方案二：

使用working set，在package explore对话框中可创建并切换到不同的working set中，也可以将working set以文件夹的形式进行组织。

2）working set的使用

打开package explore对话框，在对话框中右键创建working set

```
右键--New--Java Working set
```

![](/images/tools/eclipse/workset/newworkingset.png)

切换到working set视图，查看所有的working set（可在project与working set间不断切换）

```
package explore窗口右侧的向下三角箭头点击--Top Level Elements--Working Set/Project
```

![](/images/tools/eclipse/workset/toplevelworkingset.png)

在project中可选择不同的working set进行只展示（只有Top Level Elements为Project时才可用）

```
package explore窗口右侧的向下三角箭头点击--Select Working Set--在弹出的对话框中选择相应的working set工作空间即可
```

![](/images/tools/eclipse/workset/selectworkingset.png)