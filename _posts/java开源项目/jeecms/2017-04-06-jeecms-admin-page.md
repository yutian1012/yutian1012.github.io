---
title: 开源的门户网站jeecms--后台页面
tags: [jeecms]
---

jeecms后台页面挂接的层次分析

### 1. 后台登陆成功

后台的首页：

1）在后台登录时通过CmsAuthenticationFilter类校验用户是否有权限登录

配置文件位于：shiro-context.xml文件

```
<bean id="authcFilter" 
class="com.jeecms.core.security.CmsAuthenticationFilter" parent="adminUrlBean">
    <property name="adminIndex" value="/jeeadmin/jeecms/index.do"/>
</bean>
```

注：认证通过则转到/jeeadmin/jeecms/index.do。

2）执行WelcomeAct类的index方法

```
@RequiresPermissions("index")
@RequestMapping("/index.do")
public String index(HttpServletRequest request) {
    return "index";
}
```

注：接收处理/jeeadmin/jeecms/index.do请求

使用/WebRoot/WEB-INF/jeecms_sys/index.html页面进行组织数据，返回客户端。

3）首页加载模块（top.do和main.do两个模块）

```
<frameset rows="71,*" frameborder="0" border="0" framespacing="0">
    <frame src="top.do" name="topFrame" noresize="noresize" id="leftFrame" />
    <frame src="main.do" name="mainFrame" id="mainFrame" />
</frameset>
```

4）WelcomeAct类top方法获取页头信息

top页面：/WebRoot/WEB-INF/jeecms_sys/top.html

该页面定义了后台管理的头信息

![](/images/open/jeecms/jeecms-admin-head.png)

main正文页面：/WebRoot/WEB-INF/jeecms_sys/main.html

```
<frameset cols="210,*" frameborder="0" border="0" framespacing="0">
    <frame src="left.do" name="leftFrame" noresize="noresize" id="leftFrame" />
    <frame src="right.do" name="rightFrame" id="rightFrame" />
</frameset>
```
