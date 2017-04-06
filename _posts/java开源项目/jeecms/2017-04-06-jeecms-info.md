---
title: 开源的门户网站jeecms--目录结构信息与二次开发
tags: [jeecms]
---

### 1. 目录结构

![](/images/open/jeecms/jeecms-directory.png)

1）src目录存放java源码文件

2）WebRoot/r：web前端用到的资源，比如js, css和img

3）WebRoot/res：管理端（后台）用到的资源，比如js, css和img

4）WebRoot/thirdparty：这里放的是第三方的一些插件，比如上面提到的ckeditor，swfupload和My97DatePicker（第三方的js库）

5）WebRoot/WEB-INF/common：存用于显示上传情况，信息提示等公共页面。

6）WebRoot/WEB-INF/config：系统配置文件，如修改数据库连接，spring配置等文件。

7）WebRoot/WEB-INF/error：放一些服务器端错误提示的页面，如403错误、程序异常等

8）WebRoot/WEB-INF/ftl：管理端用到的freemarker命令定义文件

9）WebRoot/WEB-INF/ipseek：放的是IP库，用于分析IP归属地

10）WebRoot/WEB-INF/jeecms_sys：管理端HTML模板文件（ftl编写）

11）WebRoot/WEB-INF/langauges：国际化语言配置文件

12）WebRoot/WEB-INF/lucene：这个目录是lucene生成的一些所以文件

13）WebRoot/WEB-INF/t：包含前端模板HTML文件。

注：前端设计时主要关注的就是WebRoot/r和WebRoot/WEB-INF/t两个目录。

### 2. 定制化开发（后台）

如果需要进行后台的定制化开，需要在WebRoot/WEB-INF/jeecms_sys目录下进行修改。
例：创建一个模块（添加认证审核员资格查询模块）

1）需求

（1）该功能主要是完成对审核员的信息的录入（后台录入）。

（2）录入信息包括：审核员姓名，性别，出生日期，身份证号，级别，聘用方式等。

（3）前端提供一个认证审核员资格查询，查询字段为审核员姓名和身份证号

2）创建模块

在jeecms_sys目录下创建一个模块的文件夹（auditor）

提供相应的增删改查模板文件（add.html，edit.html，list.html），可参看别的页面复制并进行修改。

3）创建java类（controller，service和dao层类）

第一步：controller层，创建包：com.jeecms.cms.action.admin.auditor，并在该包下创建AuditorAct.java类。

第二步：页面位置：WEB-INF/jeecms_sys/auditor/list.html

第三步：service层，创建包：com.jeecms.cms.manager.admin.auditor，并在该包下创建接口和实现类AuditorMng.java和AuditorMngImpl.java。

第四步：dao层，创建包：com.jeecms.cms.dao.admin.auditor，并在该包下创建接口和实现类AuditorDao.java和AuditorDaoImpl.java类

第五步：在数据库中创建相应的数据模型表结构。

第六步：配置Hibernate，在src\com\jeecms\cms\entity\assist\hbm\目录下创建Auditor.hbm.xml（完成数据库表和对象的映射）。


注：具体类的实现可以参考其他模块的编写。

4）挂接java类

jeecms-servlet-admin-action.xml中配置（controller类的注入）

```
<bean id="auditorAct" class="com.jeecms.cms.action.admin.auditor.AuditorAct"/>
```

在jeecms-context.xml中配置service和dao层的实现类

```
<bean id="auditorMng" class="com.jeecms.cms.manager.admin.auditor.
impl.AuditorMngImpl"></bean>

<bean id="auditorDao" class="com.jeecms.cms.dao.admin.auditor.impl.AuditorDaoImpl"/>
```

5）挂接后台页面（add.html，edit.html，list.html）



