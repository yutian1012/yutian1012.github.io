---
title: eclipse对xml进行DTD校验
tags: [xml]
---

eclipse中使用DTD或Schema校验

1）网络问题：

在Struts, Spring, Hibernate的配置文件的时候，有时候XML编辑器的智能提示并不好用。造成这个问题的主要原因是，编辑器是从XML头部的网络地址来读取DTD或者XSD文件。

如头部命名空间的http://www.springframework.org/schema/beans/spring-beans-2.5.xsd这些文件是用来说明XML文件格式的，解析了这些文件，编辑器才能给出正确的提示。当网络状况不好或者根本没有联网的时候，是不会有正确的智能提示的。

2）获取DTD校验文件（下载校验文件）

从jar包中找出对应的DTD/Schema校验文件，拷贝到本地系统中

3）将校验文件导入到eclipse中，实现提醒

windows--preferences--xml--xml catalog，添加xml catalog Entries，选择dtd/schema文件。

![](/images/other/dtd/XMLCatalog.jpg)

参考：http://www.eclipse.org/webtools/community/tutorials/XMLCatalog/XMLCatalogTutorial.html