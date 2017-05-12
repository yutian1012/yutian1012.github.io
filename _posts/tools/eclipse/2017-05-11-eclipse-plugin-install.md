---
title: eclipse link方式离线安装插件
tags: [eclipse,tools]
---

参考：http://www.cnblogs.com/OnlyCT/p/6061134.html

实例：myeclipse离线安装Axis2插件

### 下载离线包

安装离线包：axis2-eclipse-service-plugin-1.6.2.zip和axis2-eclipse-codegen-plugin-1.6.2.zip

1）axis2-eclipse-service-plugin-1.6.2.zip插件是用来生成可以部署到webservice上的aar文件或jar文件。

2）axis2-eclipse-codegen-plugin-1.6.2.zip，This can be used to generate a WSDL file from a java class (Java2WSDL) and/or a java class file from a WSDL (WSDL2Java)，即可以有java的class文件生成wsdl文件，或由wsdl文件上传对应的java class文件。

注：WSDL（网络服务描述语言是Web Service的描述语言）。

注2：link形式的MyEclipse"非侵入式安装插件"安装方式

### 新建extplugins文件夹（统一放置插件）

1）Myeclipse本地的安装路径：E:\sjq\myeclipse2014在该目录下创建extplugins文件夹，当然，你可以在任意目录下创建

2）在extplugins文件夹下创建axis2文件夹，用来放置axis2的插件

查看axis2-eclipse-service-plugin-1.6.2.zip压缩包下的内容：

![](/images/tools/eclipse/axis2-plugin-zip.png)

发现该解压目录下只有一个plugins目录，没有features目录。将该plugins目录解压到axis2路径下。（如果压缩包下有features也一并拷贝到axis2目录下）

![](/images/tools/eclipse/axis2-plugin-copy.png)

同样将axis2-eclipse-codegen-plugin-1.6.2.zip压缩包中的plugins目录拷贝到axis2，两个的plugins就合并成一个了。

![](/images/tools/eclipse/axis2-plugin-copy2.png)

### 创建link文件指向插件文件夹

在myeclipse的dropins目录下建立一个连接文件，指向axis2目录

在dropins目录下创建axis2.link（后缀为.link）文件，编辑内容

```
path=E:/sjq/myeclipse2014/extplugins/axis2
```

这里的目录就是我们插件放置的目录。


### 重新启动myeclipse并测试

New--others，输入axis，如果能成功启动插件，证明我们安装的插件成功了！

![](/images/tools/eclipse/axis2-plugin-new.png)