---
title: 开源spagobi--源码导入
tags: [spagobi]
---

### 1. 下载源码文件

下载地址：http://forge.ow2.org/projects/spagobi/

解压该压缩包，可以看到压缩包中有很多文件夹，都是maven项目。

```
<!-- 单点登录 -->
<module>../cas</module>

<!-- 核心组件 -->
<module>../QbeCore</module>
<module>../SpagoBIDAO</module>
<module>../SpagoBIUtils</module>
<module>../SpagoBIUtilsJSON</module>
<module>../spagobi-commons-core</module>
<module>../spagobi-metamodel-core</module>
<module>../spagobi-metamodel-utils</module>
<module>../spagobi-core</module>
<module>../spagobi.birt.oda</module>  

<!-- 安全策略 -->
<module>../SpagoBIOAuth2SecurityProvider</module>
<module>../SpagoBILdapSecurityProvider</module>
<module>../SpagoBILiferaySecurityProvider</module> 

<!-- web项目-提供管理平台 -->
<module>../SpagoBIProject</module>

<!-- 分析引擎 -->
<module>../SpagoBICockpitEngine</module> 
<module>../SpagoBIQbeEngine</module>
<module>../SpagoBIAccessibilityEngine</module> 
<module>../SpagoBIBirtReportEngine</module> 
<module>../SpagoBIChartEngine</module> 
<module>../SpagoBICommonJEngine</module> 
<module>../SpagoBIConsoleEngine</module> 
<module>../SpagoBIDataMiningEngine</module> 
<module>../SpagoBIGeoEngine</module> 
<module>../SpagoBIGeoReportEngine</module> 
<module>../SpagoBIJasperReportEngine</module> 
<module>../SpagoBIJPivotEngine</module> 
<module>../SpagoBIMobileEngine</module> 
<module>../SpagoBINetworkEngine</module>
<module>../SpagoBISocialAnalysis</module>
<module>../SpagoBITalendEngine</module>
<module>../SpagoBIWhatIfEngine</module>
```

### 2. eclipse中导入源码

压缩包中的maven项目很多，但真正使用的并不是所有的项目，导入时只需要导入需要的maven项目即可。

1）导入spagobi-parent项目

上面的导入方法是单独导入某个项目，要想让spagoBI正常的运行，还需要导入相关的引擎。因此直接导入压缩包中的spagobi-parent项目，就可以将所有相关的项目都导入到eclipse中了。

报错：

```
: is an invalid character in resource name 'it.eng.spagobi:spagobi-parent'.
```

注：这种方式无法成功的将项目导入，只能分别导入。

2）导入前先下载项目的依赖

定位目录到压缩包的spagobi-parent目录下，执行命令

```
mvn install -Dmaven.test.skip=true
```

注：执行该命令，会将相关的依赖都安装到mvn仓库中，导入单个项目后就不会因为依赖文件无法下载而报错。

3）导入下面的项目：（import--existing maven projects）

SpagoBIProject、ChartEngline、CockpitEngline、WhatifEngline（3个引擎）、QbeCore、SpagoBIDAO、SpagoBIUtils、SpagoBIUtilsJSON。

注：将设置项目的编码为utf-8，这些项目是比较重要的。

### 3. 创建数据库

下载mysql-dbscripts-5.2.0_24032016.zip

在mysql中创建数据库spagoBI，并执行压缩包下的sql。安装mysql数据库，并创建数据库spagobi，运行sql创建数据表。

```
mysql-dbupgradescript-2.1-to-2.2：是系统升级时使用的补丁sql，这里直接运行
MySQL_create.sql：创建数据表
MySQL_create_quartz_schema.sql：创建定时器数据表
MySQL_create_social.sql：创建社交相关的数据表
```

### 4. 测试连接

1）数据源配置，在tomcat的server.xml中配置数据源

```
<Resource name="jdbc/spagobi" auth="Container"
          type="javax.sql.DataSource" driverClassName="com.mysql.jdbc.Driver"
          url="jdbc:mysql://localhost:3306/spagobi"
          username="root" password="root" maxActive="20" maxIdle="10"
          maxWait="-1"/>
```

注：在<GlobalNamingResources>标签下添加数据源

2）测试mysql运行

将D:\All-In-One-SpagoBI-5.2.0\lib\mysql-connector-java-5.0.8-bin.jar文件拷贝到tomcat的\lib目录下，启动tomcat。

并将D:\All-In-One-SpagoBI-5.2.0\webapps\SpagoBI拷贝到tomcat的webapps目录下，即发布该项目到tomcat。

启动tomcat，报错

```
Caused by: com.mysql.jdbc.exceptions.MySQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'OPTION SQL_SELECT_LIMIT=1' at line 1
```

解决方法

升级mysql驱动jar包，mysql-connector-java-5.1.27.jar拷贝到D:\All-In-One-SpagoBI-5.2.0\lib下，替换掉mysql-connector-java-5.0.8-bin.jar.

3）SpagoBI项目

D:\All-In-One-SpagoBI-5.2.0\webapps\SpagoBI其实是SpagoBIProject发布后的项目，如果直接使用All-In-One_spagoBI下的项目，默认使用的是HSQLDB的连接方式，因此修要修改成mysql的连接。

启动tomcat，错误信息：

```
Caused by: it.eng.spagobi.utilities.exceptions.SpagoBIRuntimeException: Cannot configure cache: Data source for writing is not defined. Please select one in the data sources definition panel.
```

注：数据源问题，使用的数据库连接驱动错误，修改为mysql的数据库连接驱动。

修改：

hibernate.cfg.xml，jbpm.hibernate.cfg.xml，quartz.properties

```
hibernate.cfg.xml文件
<property name="hibernate.dialect">org.hibernate.dialect.HSQLDialect</property>
改为：
<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

jbpm.hibernate.cfg.xml文件
<property name="hibernate.dialect">org.hibernate.dialect.HSQLDialect</property>
改为：
<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

quartz.properties文件
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.HSQLDBDelegate
改为：
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
```

启动tomcat，错误信息：

```
Caused by: it.eng.spagobi.utilities.exceptions.SpagoBIRuntimeException: Cannot configure cache: Data source for writing is not defined. Please select one in the data sources definition panel.
```

猜测这里的数据源应该是类似样例数据，如foodmart数据库（hsqldb提供）。

登录系统，创建数据源：

![](/images/open/spagobi/spagobi-datasource.png)

在系统中创建数据源，之后再次重启就不报错了，和之前猜测的相符。

4）环境变量设置

```
it.eng.spagobi.commons.utilities.SpagoBIUtilities.readJndiResource: javax.naming.NameNotFoundException: Name [spagobi_resource_path] is not bound in this Context. Unable to find [spagobi_resource_path].
```

参考D:\All-In-One-SpagoBI-5.2.0\conf\server.xml，将对应的环境变量配置到运行的tomcat的配置文件中

```
<Environment name="spagobi_resource_path" type="java.lang.String" value="${catalina.base}/resources"/>  
<Environment name="spagobi_sso_class" type="java.lang.String" value="it.eng.spagobi.services.common.FakeSsoService"/>
<Environment name="spagobi_service_url" type="java.lang.String" value="http://localhost:8080/SpagoBI"/>   
<Environment name="spagobi_host_url" type="java.lang.String" value="http://localhost:8080"/>
```

代码中获取

```
Context ctx = new InitialContext();
//jndiName--java://comp/env/spagobi_resource_path
System.out.println("*****************************************"+jndiName);
value = (String) ctx.lookup(jndiName);
//D:\Program Files\apache-tomcat-7.0.59/resources
System.out.println("*****************************************"+value);
```

文件位置：configs.xml文件中使用了spagobi_resource_path作为jndi的查找路径

```
//文件位置
/SpagoBIProject/src/main/resources/it/eng/spagobi/commons/initializers/metadata/config/configs.xml

//代码：
<CONFIG label="INDEX_INITIALIZATION.jndiResourcePath" name="INDEX_INITIALIZATION.jndiResourcePath" description="" isActive="true" valueCheck="java://comp/env/spagobi_resource_path" valueType="STRING" category="SEARCH_ENGINE"/>
```

拷贝resources文件夹

将All-In-One-SpagoBI-5.2.0_11042016压缩包中的resources文件夹拷贝到D:\Program Files\apache-tomcat-7.0.59（tomcat的根目录下，实际运行的tomcat）


### 5. 项目部署

1）部署SpagoBIProject项目

在eclipse中导入SpagoBIProject，并修改相应的mysql连接文件，设置数据源。将项目部署到tomcat上，然后启动。

错误信息：

```
java.lang.NoClassDefFoundError: bsh/EvalError
```

修改方法：jbpm.hibernate.cfg.xml文件

```
删除/隐藏文件中的语句
<mapping resource="org/jbpm/graph/action/Script.hbm.xml"/>
```

修改方法2：

```
<dependency>
    <groupId>bsh</groupId>
    <artifactId>bsh</artifactId>
    <version>1.3.0</version>
</dependency>
```

2）部署运行

部署所有以engine结尾的项目以及SpagoBIProject项目和SpagoBISocialAnalysis项目。也可以单独部署需要的分析引擎，不用全部部署，这里作为测试全部部署。

注：在部署前需要修改数据库连接方式。

```
修改属性文件中的数据库类型
SpagoBICockpitEngine（hibernate.cfg.xml），
SpagoBIDataMiningEngine(hibernate.cfg.xml)，
SpagoBIMobileEngine(hibernate.cfg.xml)，
SpagoBIProject（hibernate.cfg.xml，jbpm.hibernate.cfg.xml，quartz.properties）
SpagoBISocialAnalysis(hibernate.cfg.xml，quartz.properties)
SpagoBIGeoReportEngine（hibernate.cfg.xml）
```

3）访问服务

访问地址：http://localhost:8080/SpagoBI/ 

用户名：biadmin，秘密：biadmin
