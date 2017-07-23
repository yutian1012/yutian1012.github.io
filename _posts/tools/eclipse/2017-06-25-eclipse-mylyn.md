---
title: mylyn工具
tags: [eclipse,tools]
---

参考：http://www.blogjava.net/alwayscy/archive/2008/06/15/208022.html

### 使用场景

相信很多人都有过这样的经验，改一个东西可能就几分钟，但找到在哪改、会影响到什么地方，却要花半小时。有了这个工具，让我们在非常大的项目里，在文件和代码的海洋里能马上找到所要关注的部分。有的人说，我有CTRL+SHIFT+T，可是你能记住几年前一个项目里的类名吗？而查阅文字描述的任务却要容易得多。 

### 作用

把任务列表与具体的代码联系到了一起。你只要激活一个任务，之相关的所有文件、函数将被突出的显示在ECLIPSE界面的每个"角落"――Package Explorer，Open Type, Open Resource，Debug View……

### Mylyn是如何做到任务与代码的关联呢？

你唯一要做的就是，在完成一个编码任务前，激活相应的任务！这样，随后你的编辑、访问各种元素的操作都被Mylyn记录，它会根据你的访问频率分析相关程度。当你的任务成百上千，或者你过一段时间再回头来修改代码时，只要激活相应的任务，它就会自动将相关的文件窗口打开，并在各种查找、显示界面里根据当初的记录突出显示相应元素。

首先，新建一个任务，激活任务，那么在激活任务后修改的所有代码都会与这个任务建立关系。