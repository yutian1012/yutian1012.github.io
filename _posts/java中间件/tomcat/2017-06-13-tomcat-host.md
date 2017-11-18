---
title: tomcat配置虚拟主机
tags: [tomcat]
---

参考：http://blog.csdn.net/springsky_/article/details/7052419

1）修改tomcat的service.xml文件

文件位置：%TOMCAT_HOME%/conf/server.xml

2）配置host节点

在Engine节点中添加Host节点，设置虚拟主机

```
<Host name="www.abc.net" appBase="webapps" unpackWARs="true" autoDeploy="true">      
    <Alias>abc.com</Alias>    
    <Context path="/hello" docBase="/var/www/html/hello" debug="0" privileged="true"/>   
</Host>  
```

注：Host节点代表为一个虚拟主机，name表示需要访问的域名，这个域名是已经注册的域名。另外，一个虚拟机可以使用多个域名，在Host节点下配置Alias映射多个域名。

注2:appBase表示指定的项目父路径，在当前的路径下可以存放多个项目。在该目录下放置的所有项目，除了ROOT目录下的项目是一级域名访问的项目，其他项目都可以使用二级域名访问。

注3：使用Context也可以配置二级域名，其中path指定了二级域名的访问路径。

3）二级域名的配置方式

方式一：将项目放置到appBase指定的路径下，并使用相应的文件夹名称作为二级域名的访问路径。

方式二：在Host节点下配置Context节点，设置path（二级域名访问路径）和docBase（项目目录）。

### 实例

系统中使用一个tomcat配置了两个项目，项目A和项目B，如果进行域名的绑定？

1）项目背景

原来系统只有一个项目，并且外部做了域名解析，会将域名访问定位到该服务上，因此tomcat服务器的server.xml配置不需要太多改动，只需要改动端口8080为80即可。项目直接放置到webapps目录下，且命名为ROOT（删除原来的ROOT）。

Host节点配置（使用默认值，不需要更改）

```
<Host name="localhost"  appBase="webapps"  unpackWARs="true" autoDeploy="true">
...
```

2）增加新的项目

在%TOMCAT_HOME%目录下创建abc文件夹，并将项目拷贝到abc文件夹下，重命名项目文件名为ROOT

```
<Host name="www.abc.com" appBase="abc" unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="abc_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

注：Valve设置了日志文件的输出文件名及输出格式。

3）两个项目的组织方式

项目1：%TOMCAT_HOME%/webapps/ROOT

项目2：%TOMCAT_HOME%/abc/ROOT

4）配置两个项目的webapp.root

log4j日志模块中使用的配置信息，修改web.xml文件的webAppRootKey值，防止使用相同的value。

```
<context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>webapp.root</param-value>
</context-param>
```

项目A修改为：

```
<context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>webapp.rootA</param-value>
</context-param>
```

项目B修改为：

```
<context-param>
    <param-name>webAppRootKey</param-name>
    <param-value>webapp.rootB</param-value>
</context-param>
```