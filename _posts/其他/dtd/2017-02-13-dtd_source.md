---
title: xml中设置DTD-来源
tags: [other]
---

DTD文档类型定义(Document Type Definition)是一套为了进行程序间的数据交换而建立的关于标记符的语法规则。

DTD文档的来源主要分为三种：文档内部定义，本地定义单独文件，公共引用。

### 1. 内部DTD

内部dtd放在格式数据的xml里面。

1）语法

```
<!DOCTYPE 根元素名 [
   <!ELEMENT 元素名 (元素类型定义)>
]>
```

注：注意的是在元素名后面一定要有空格，否则就不是格式良好的，

2）DTD与xml文档放置在一起：

```
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<!DOCTYPE poem [
  <!ELEMENT poem (title,author,line+)>
  <!ELEMENT title (#PCDATA)>
  <!ELEMENT author (#PCDATA)>
  <!ELEMENT line (#PCDATA)>
]>
<poem>
 <title>静夜思</title>
 <author>李白</author>
 <line>床前明月光,</line>
 <line>疑事地上霜.</line>
 <line>举头望明月,</line>
 <line>低头思故乡.</line>
</poem>
```

注：如果把DTD放在xml文档内部，一方面会带来xml文档变大，一些程序可能不需要DTD信息；另一方面不利于DTD共用，也许会有不同的xml文档共用这个DTD。

### 2. 外部DTD

将DTD文档单独定义成一个独立的文档，xml引用该独立文档即可。

1）定义外部DTD的语法：

```
<!DOCTYPE  根元素名 SYSTEM "外部DTD文件的URI">
```

注：SYSTEM表示DTD文件是私有的，外部DTD文档的uri需要使用引号括起来，引号不能少。

2）定义DTD文档

```
<?xml version="1.0" encoding="UTF-8"?>
<!ELEMENT poem (title,author,line+,commet)>
<!ELEMENT title (#PCDATA)>
<!ELEMENT author (#PCDATA)>
<!ELEMENT line (#PCDATA)>
<!ELEMENT commet  (#PCDATA)>
```

3）相同目录下定义xml文档

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<?xml-stylesheet href="simple1.css" type="text/css" ?>
<!-- 外部DTD -->
<!DOCTYPE poem SYSTEM "poem.dtd">
<poem>
 <title>静夜思</title>
 <author>李白</author>
 <line>床前明月光,</line>
 <line>疑事地上霜.</line>
 <line>举头望明月,</line>
 <line>低头思故乡.</line>
 <commet>李白是中国最伟大的诗人！</commet>
</poem>
```

### 3. 引用公共的DTD文档

1）语法：

```
<!DOCTYPE  根元素名 PUBLIC "DTD的名称" "外部DTD文件的URI">
```

注：PUBLIC表示DTD文件是公共的，PUBLIC之后，还多了一个DTD的名称(引号不能少)。

如，taglib的xml引用DTD文档

```
<!DOCTYPE taglib PUBLIC "-//Sun Microsystems, Inc.//DTD JSP Tag Library 1.2//EN" "http://Java.sun.com/dtd/web-jsptaglibrary_1_2.dtd">
```

2）分析一下这个外部DTD声明：

关键字DOCTYPE，PUBLIC。

根元素名：taglib。所以每一个标签库定义文件都是以taglib为根元素的，否则就不会验证通过。

"-//Sun Microsystems, Inc.//DTD JSP Tag Library 1.2//EN"，这是公共DTD的名称。首先它是以"-"开头的，表示这个DTD不是一个标准组织制定的。（如果是ISO标准化组织批准的，以"ISO"开头，如果不是ISO的标准化组织批准的，以"+"开头。）。接着就是双斜杠"//"，跟着的是DTD所有者的名字，很明显这个DTD是sun公司定的。接着又是双斜杠"//"，然后跟着的是DTD描述的文档类型，可以看出这份DTD描述的是jsp标签库1.2版本的格式。再跟着的就是"//"和ISO 639语言标识符。

"http://java.sun.com/dtd/web-jsptaglibrary_1_2.dtd"，表示这个DTD的位置

参考：http://blog.csdn.net/centre10/article/details/5932205

### 4. 自己编写DTD校验xml

XML引用DTD使用相对路径

```
<!DOCTYPE poem SYSTEM "../poem.dtd">
```

注：该dtd位于xml文件的上层目录中。

### 5. eclipse中使用DTD或Schema校验

1）问题：

在Struts, Spring, Hibernate的配置文件的时候，有时候XML编辑器的智能提示并不好用。造成这个问题的主要原因是，编辑器是从XML头部的网络地址来读取DTD或者XSD文件，比如头部命名空间的http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  这些文件是用来说明XML文件格式的，解析了这些文件，编辑器才能给出正确的提示。当网络状况不好或者根本没有联网的时候，是不会有正确的智能提示的。

2）获取DTD校验文件

从jar包中找出对应的DTD/Schema校验文件，拷贝到本地系统中

3）将校验文件导入到eclipse中，实现提醒

windows--preferences--xml--xml catalog，添加xml catalog Entries，选择dtd/schema文件。

![](/images/other/dtd/XMLCatalog.jpg)


参考：http://www.eclipse.org/webtools/community/tutorials/XMLCatalog/XMLCatalogTutorial.html
