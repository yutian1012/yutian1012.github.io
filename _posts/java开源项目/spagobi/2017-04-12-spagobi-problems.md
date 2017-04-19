---
title: 开源spagobi--常见问题
tags: [spagobi]
---

### 1. 抛出异常：Out of memory

解决方法：

方式一：在catalina.bat中设置下面变量

```
set JAVA_OPTS= %JAVA_OPTS%  -Xms1024m -Xmx1024m -XX:MaxPermSize=512m
```

方式二：在myeclipse中设置

```
-Xms1024m -Xmx1024m -XX:MaxPermSize=512
```

![](/images/open/spagobi/myeclipse-jvm-mem.png)

### 2. 日志信息错误

错误信息：

```
Caused by: java.lang.NoClassDefFoundError: Lorg/apache/log4j/Logger;
```

拷贝相关jar包到tomcat/lib下，根据All-In-One-SpagoBI-5.2.0_11042016.zip中解压后的tomcat下的lib包中的jar拷贝相应的jar包。

拷贝如下jar包：

```
commons-logging-1.1.1.jar
commons-logging-api-1.1.jar
concurrent.jar--提供标准化、高效的版本的实用的类，用于并行的Java程序
foo-commonj.jar--foo-commonj.jar some required libraries for JDBC Drivers
geronimo-commonj_1.1_spec-1.0.jar--Java API for concurrent programming of EJBs and Servlets
```

### 3. User.hbm.xml文件找不到

查看tomcat/log/SpagoBI.txt日志文件

```
org.hibernate.MappingNotFoundException: resource: org/jbpm/identity/User.hbm.xml not found。
```

修改jbpm.hibernate.cfg.xml文件

```
<!-- identity mappings (begin) -->
    <mapping resource="org/jbpm/identity/User.hbm.xml"/>
    <mapping resource="org/jbpm/identity/Group.hbm.xml"/>
    <mapping resource="org/jbpm/identity/Membership.hbm.xml"/>
<!-- identity mappings (end) -->
```

注：这是jBPM自带的用户组织系统，被用来分配流程参与者的。我们的建库脚本中并没有这3张表（当然，你有兴趣的话也可以自己建这三张表），所以请把这3行注释掉！则jBPM将会使用Tomcat的用户系统来分配参与者。

Jbpm-identity.jar包在jbpm.3.x.jar版本中存在（压缩包下的/build目录下），之后的版本（4.x）中已经找不到该文件了。

注：在jbpm-identity.jar的org\jbpm\identity目录下存在User.hbm.xml（解压缩该jar包查看），hibernate能够根据该文件自动创建表。

### 4. icu4j错误

icu4j.jar用于增强对多语言的支持。

使用All-In-One-SpagoBI-5.2.0_11042016.zip下载包运行程序报错：

```
Unable to process Jar entry [com/ibm/icu/impl/data/LocaleElements_zh__PINYIN.class] from Jar [jar:file:/ G:/spagoBi/All-In-One-SpagoBI-5.2.0_11042016/All-In-One-SpagoBI-5.2.0
/webapps/SpagoBIQbeEngine/WEB-INF/lib/icu4j-2.6.1.jar!/] for annotations
```

解决方案1：

```
I solved the same problem by simply replacing the icu4j-2.6.1.jar with latest jar
```

下载地址：http://www.icu-project.org/download/4.0.html

解决方案2：

升级SpagoBIQbeEngine项目中pom中的jaxen至1.1.4或更高版本，因为jaxen依赖XOM，XOM依赖icu4j。但jaxen自1.1.4版本开始已经不依赖XOM.jar了，自然也不需要icu4j了，没有这个包自然就不用担心它报错了。

```
mvn dependency:tree --查看依赖

jaxen:jaxen:jar:1.1:compile
|  +- jdom:jdom:jar:1.0:compile
|  +- xerces:xercesImpl:jar:2.6.2:compile
|  \- xom:xom:jar:1.0:compile
|     +- xerces:xmlParserAPIs:jar:2.6.2:compile //与icu是平级的
|     \- com.ibm.icu:icu4j:jar:2.6.1:compile
```



