---
title: 开源spagobi--安装和使用
tags: [spagobi]
---

### 1. 下载安装包

1）下载：All-In-One-SpagoBI-5.2.0_11042016.zip

![](/images/open/spagobi/spagobidownload.png)

2）检查本地安装的java版本（必须1.7及以上）

```
java -version
```

注：设置环境变量JAVE_HOME

3）运行测试

运行文件

```
All-In-One-SpagoBI-5.2.0\bin\SpagoBIStartup.bat
```

访问地址：http://localhost:8080/SpagoBI

用户名和密码都是：biadmin

3）用户信息

用户名密码：and login using the user biadmin and password biadmin.

Note:  By default, there are various other users e.g  bitest, bimodel, bidev, biuser  with password being the same as the username but we will ignore these other users at this point.

### 2. 分析运行过程

1）查看运行批处理文件SpagoBIStartup.bat文件

```
if "%OS%" == "Windows_NT" setlocal

set CURRENT_DIR=%cd%
if not "%CATALINA_HOME%" == "" goto gotHome
set CATALINA_HOME=%CURRENT_DIR%
if exist "%CATALINA_HOME%\bin\catalina.bat" goto okHome
cd ..
set CATALINA_HOME=%cd%
cd %CURRENT_DIR%

:gotHome
if exist "%CATALINA_HOME%\bin\catalina.bat" goto okHome
echo The CATALINA_HOME environment variable is not defined correctly
echo This environment variable is needed to run this program
goto end

:okHome
set EXECUTABLE=%CATALINA_HOME%\bin\catalina.bat

if exist "%EXECUTABLE%" goto okExec
echo Cannot find %EXECUTABLE%
echo This file is needed to run this program
goto end

:okExec

set CMD_LINE_ARGS=

:setArgs
if ""%1""=="""" goto doneSetArgs
set CMD_LINE_ARGS=%CMD_LINE_ARGS% %1
shift
goto setArgs
:doneSetArgs

set CD=%cd%
cd %CATALINA_HOME%\database
start "HSQLDB" /max cmd.exe /c start.bat

cd %CD%
call "%EXECUTABLE%" start %CMD_LINE_ARGS%

:end
```

注：代码判断CATALINA_HOME环境变量是否配置，如果配置了则跳转到CATALINA_HOME下运行。

```
if not "%CATALINA_HOME%" == "" goto gotHome
```

因此，如果系统中配置了CATALINA_HOME可能无法正常运行成功，而会去运行tomcat下的web应用，不会运行spagobi。

解决方法，将上面的判断代码隐掉（rem注释代码）。

sql脚本的运行（脚本文件位于压缩包的database目录下）

```
set CD=%cd%
cd %CATALINA_HOME%\database
start "HSQLDB" /max cmd.exe /c start.bat
```

注：start命令是启动一个新的运行窗口，执行cmd.exe命令启动命令提示符窗口，

注2：/c start.bat表示执行完start.bat后关闭窗口

如开启一个花板程序

```
start mspaint
```

查看database下的start.bat文件

```
java -Xms1024m -Xmx1024m -XX:MaxPermSize=512m -cp ./hsqldb1_8_0_2.jar org.hsqldb.Server -database.0 ./foodmart -dbname.0 foodmart 
```

注：指定对应的数据库文件foodmart.*（包括foodmart.log，foodmart.properties，foodmart.script等）。

2）运行过程

首先设置CATALINA_HOME，找到执行文件catalina.bat

然后启动hsqldb服务，即执行HSQLDB，为应用提供数据库支持

最后运行catalina启动服务

### 3. spagoBI的结构

Spagobi应用是以核心加引擎的模型来组织应用的。spagobi发行包中包含多个引擎，但是在实际使用中已去除多个引擎，只保留了SpagoBIJPivotEngine和SpagoBITalendEngine引擎。如果需要其他引擎可以下载并安装。

1）引擎与分析

It consists of several engines and analytical areas. The engines includes SpagoBIBirtReportEngine, SpagoBIJPivotEngine and SpagoBIHighChartsEngine. In total there are around 21 engines and the complete list can be found at http://www.spagoworld.org/xwiki/bin/view/SpagoBI/AnalyticalEngines. We will concentrate on reporting using the SpagoBIBirtReportEngine（报表使用引擎）,  OLAP using the SpagoBIJPivotEngine（OLAP主要使用引擎） and Charts using the SpagoBIHighChartsEngine（图表主要使用引擎）. 

2）SpagoBI Server（业务智能平台）

This is the actual business intelligence platform that offers all the core and  analytical functionalities. It is also where we will be hosting all reports created using BIRT. Click on All-In-One-SpagoBI-3.3-01242012.zip to download the SpagoBI Server as illustrated below. 

3）SpagoBI Studio （工作空间）

We will need the SpagoBI studio to create BIRT reports（创建报表）.  BIRT is an eclipse based  business intelligence and reporting tool  and the acronyms（缩略词） stand for Business Intelligence and Reporting Tools. Download SpagoBI Studio by clicking on SpagoBIStudio_3.3_win_20120120.zip as illustrated below. 

BIRT（Business Intelligence and Reporting Tools）.

### 4. 遇到的问题

1）问题1

下载最新的SpagoBI后，执行startup.bat，一闪而过（可能是内存分配不够）。

在cmd中执行startup.bat和shutdown.bat，发现错误：

```
Error occurred during initialization of VM
Could not reserve enough space for object heap
Error: Could not create the Java Virtual Machine.
```

解决方法：

查看catalina.bat文件，发现设置的内容太大，不能分配这么多的内容。

文件设置值：

```
set JAVA_OPTS= %JAVA_OPTS% -Xms1024m -Xmx1024m -XX:MaxPermSize=512m
```

修改为：

```
set JAVA_OPTS= -server -Xms800m -Xmx800m -XX:MaxNewSize=256m
```

2）问题2

启动报错：（查看tomcat下的日志文件catalina.2016-04-21.log）

```
Unable to process Jar entry [com/ibm/icu/impl/data/LocaleElements_zh__PINYIN.class] from Jar 
jar:file:/D:/All-In-One-SpagoBI-5.2.0/webapps/SpagoBIQbeEngine/WEB-INF/lib/icu4j-2.6.1.jar!/] for annotations
```

下载icu4j.3.6.1.jar包，替换掉该icu4j-2.6.1.jar

```
<dependency>
    <groupId>com.ibm.icu</groupId>
    <artifactId>icu4j</artifactId>
    <version>3.6.1</version>
</dependency>
```

