---
title: 开源spagobi--ConfigServlet代码分析（重点）
tags: [spagobi]
---

### 1. 初始化

web.xml文件中配置

```
<servlet>
    <servlet-name>ConfigServlet</servlet-name>
    <servlet-class>it.eng.spago.configuration.ConfigServlet</servlet-class>
    <init-param>
        <param-name>AF_CONFIG_FILE</param-name>
        <param-value>/WEB-INF/conf/master.xml</param-value>
    </init-param>
    <load-on-startup>0</load-on-startup>
</servlet>
```

注：AF_CONFIG_FILE指定了配置文件的路径。

```
if (afRootPath == null) {
    ConfigSingleton.setConfigurationCreation(new StreamCreatorConfiguration());
    ConfigSingleton.setRootPath(config.getServletContext().getRealPath(""));
}
```

注：使用StreamCreatorConfiguration进行配置文件的读取操作

ConfigServelt类调用InitializerManager.init();进行初始化操作。

### 2. InitializerManager.init方法

调用ConfigSingleton.getInstance()方法会调用到ConfigSingleton类的构造方法。会使用StreamCreatorConfiguration类读取配置文件，返回SourceBean。

StreamCreatorConfiguration解析xml过程：

由SourceBeanContentHandler来处理xml文件，并返回sourceBean对象。xml中的主标签是最顶层（sourceBean的name属性值），其直接子标签创建新的sourceBean对象，name为子标签名称，子标签属性放置在_attributes属性中（新建的sourceBean对象），类型为SourceBeanAttribute，最后将子标签（新建的sourceBean对象）设置到父标签的_attributes属性中。因此，通过最外层的SourceBean对象就能实现遍历所有的子标签（sourceBean对象）及其属性值。

### 3. SourceBean对象

解析xml文件，将其属性封装到_attributes集合对象中，并提供了对集合的遍历等访问和设置操作。

### 4. ConfigSingleton对象

该对象继承了SourceBean。

ConfigSingleton.getInstance()单例方法实现解析配置文件过程，项目的主配置文件为master.xml，master.xml文件中又指定了其他配置文件的path，具体过程：

1）解析master.xml封装成sourcebean

ConfigSingleton的构造函数首先会解析master.xml，将xml中的属性封装以SourceBeanAttribute的形式封装到_attributes集合中。

2）遍历_attributes集合中CONFIGURATOR属性

CONFIGURATOR标签为master.xml文件中的MASTER标签下的子标签，该标签中设置了其他配置文件的path。每一个配置文件都会解析为一个ConfigSingleton对象，做为调用对象（即父对象ConfigSingleton）的_attributes集合元素。因此最终得到的ConfigSingleton对象的_attributes集合中包含多个名为CONFIGURATOR的SourceBean和多个ConfigSingleton（解析path路径下的配置文件xml获取的对象）。

![](/images/open/spagobi/source/configSingleton.png)

### 5. master.xml主配置文件

```
<MASTER>
...
<!-- webapp --> 
<CONFIGURATOR path="/WEB-INF/conf/webapp/messages.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/modules.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/pages.xml" />
<CONFIGURATOR path="/WEB-INF/conf/webapp/presentation.xml" />
...
</MASTER>
```

注：这里的注释表示下面的配置文件中的信息都属于该模块，即为该模块提供服务。

注2：配置文件中的类信息类似于spring中管理的bean，用于为系统提供服务。类的获取通过configure.getFilteredSourceBeanAttribute(...)获取配置信息，SourceBean对象。