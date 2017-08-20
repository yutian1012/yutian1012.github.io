---
title: 导出word的6中方案
tags: [export]
---

### java导出word大致有6种解决方案：

1）jacb

Jacob是Java-COM Bridge的缩写，它在Java与微软的COM组件之间构建一座桥梁。使用Jacob自带的DLL动态链接库，并通过JNI的方式实现了在Java平台上对COM程序的调用。DLL动态链接库的生成需要windows平台的支持。该方案只能在windows平台实现，是其局限性。

2）POI

Apache POI包括一系列的API，它们可以操作基于MicroSoft OLE 2 Compound Document Format的各种格式文件，可以通过这些API在Java中读写Excel、Word等文件。他的excel处理很强大，对于word还局限于读取，目前只能实现一些简单文件的操作，不能设置样式。

3）java2word

Java2word是一个在java程序中调用 MS Office Word 文档的组件（类库）。该组件提供了一组简单的接口，以便java程序调用他的服务操作Word 文档。 这些服务包括： 打开文档、新建文档、查找文字、替换文字，插入文字、插入图片、插入表格，在书签处插入文字、插入图片、插入表格等。填充数据到表格中读取表格数据 ，1.1版增强的功能： 指定文本样式，指定表格样式。如此，则可动态排版word文档。是一种不错的解决方案。

4）iText

iText是著名的开放源码的站点sourceforge一个项目，是用于生成PDF文档的一个java类库。通过iText不仅可以生成PDF或rtf的文档，而且可以将XML、Html文件转化为PDF文件。功能强大。

5）JSP

JSP输出样式，该方案实现简单，但是处理样式有点缺陷，简单的导出可以使用。

参考：http://blog.csdn.net/chenzenan/article/details/6439984

6）xml方式

用XML做就很简单了。Word从2003开始支持XML格式，大致的思路是先用office2003或者2007编辑好word的样式，然后另存为xml，将xml翻译为FreeMarker模板，最后用java来解析FreeMarker模板并输出Doc。经测试这样方式生成的word文档完全符合office标准，样式、内容控制非常便利，打印也不会变形，生成的文档和office中编辑文档完全一样。