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

14）WebRoot/u：系统上传附件的存放位置

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

注：以上步骤即可完成后台的编辑和查询，下面介绍前端功能查看

### 3. 前端提供一个认证审核员资格查询

前端提供查询功能

![](/images/open/jeecms/jeecms-front-add.png)

1）在前端的导航栏挂接查询功能连接

主要分为两种方式，方式一通过创建栏目的方式，方式二通过在导航栏添加链接的方式。

鉴于我们这里只提供一个前端的查询功能，可以使用一个单页栏目模型创建栏目，在栏目中直接设置查询内容。

2）实现查询功能

页面表单提交查询

```
<form action="${base}/auditorcersearch.jspx" target="_self" id="formauditorcersear">
    <h5 class="tit">姓名：</h5>
    <input name="auditorname" value="[#if auditorname??]${auditorname}[/#if]" type="text" id="auditornameKey"  maxlength="50" placeholder="姓名"/>
    
    <h5 class="tit">证书编号：</h5>
    <input name="certificateNum"  value="[#if certificateNum??]${certificateNum}[/#if]" type="text" id="certificateNumKey"  maxlength="50" placeholder="证书编号"/>
    
    <a href="javascript:searchauditorcer();"  class="con_more">搜索</a>
    <div class="ser_msg" style="color:green;">查询时请输入准确的名称或证书编号!</div>
    <input type="hidden" name="tag" value="auditorcertag">
</form>
```

在jeecms-servlet-front-action.xml中设置查询处理类

```
<bean id="auditorSearch" class="com.jeecms.cms.action.front.AuditorSearch"/>
```

AuditorSearch类

```
@RequestMapping(value = "/auditorcersearch*.jspx", method = RequestMethod.GET)
public String searchAuditorCer(HttpServletRequest request,
       HttpServletResponse response, ModelMap model) {
```

结果集显示：

结果集的显示有两种方式，方式一后台连接处理获取查询结果的list集合，前台使用freemarker的list标签命令显示list集合信息。方式二，使用自定义标签显示显示，将相应的参数设置到自定义标签上。这里使用自定义标签实现。

```
[@cms_auditorcer_page  count='15' auditorname=auditorname certificateNum=certificateNum sysPage='2' orderBy='2' ]
    <table class="check_list m_t">
        <thead>
            <tr>
                <th>姓名</th>
                <th>性别</th>
                <th>证书编号</th>
                <th>培训机构</th>
                <th>发证日期</th>
            </tr>
        </thead>
        <tbody>
            [#if tag_pagination.list?size==0]
            <tr><td colspan="5" class="t5">无符合条件的查询结果!</td></tr>
            [/#if]
            [#list tag_pagination.list  as content]
                <tr>
                    <td class="t1" style="width:10%">${content.username!}</td>
                    <td class="t2" style="width:10%">[#if content.sex?? && content.sex=='1']男[/#if][#if content.sex?? && content.sex=='0']女[/#if]</td>
                    <td class="t3" style="width:20%">${content.certificateNum}</td>
                    <td class="t4" style="width:30%">${content.trainOrg}</td>
                    <td class="t5" style="width:10%">[#if content.cerDate??]${content.cerDate?string('yyyy-MM-dd')}[/#if]</td>
                </tr>
            [/#list][#rt/]
        </tbody>
    </table>
[/@cms_auditorcer_page]
```

注：这里的自定义标签参数auditorname和certificateNum都是在AuditorSearch类searchAuditorCer方法中将表单提交的值设置到ModelMap中，然后在ftl模板中就可以从ModelMap中获取参数值，进而使用自定义标签来执行查询，并显示结果。