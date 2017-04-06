---
title: 开源的门户网站jeecms--安装
tags: [jeecms]
---

### 1. 在线安装

1）安装

需要下载系统安装包，解压后放置到tomcat的webapps目录下，重启tomcat即可使用安装向导完成安装。

2）安装过程分析

安装文件位于项目的根目录的install文件夹中。

启动tomcat后，访问系统：http://localhost:8080/jeecms

会访问到install.html页面，进入安装向导。因为在web.xml中配置的欢迎页面为install.html

```
<servlet>
    <servlet-name>Install</servlet-name>
    <servlet-class>com.jeecms.cms.InstallServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>Install</servlet-name>
    <url-pattern>/install/install_setup.svl</url-pattern>
</servlet-mapping>
<welcome-file-list>
    <welcome-file>install.html</welcome-file>
</welcome-file-list>
```

注：可以看到install的处理时交由InstallServlet来处理的。

3）运行安装

（1）license安装许可（/install/index.html）,下一步表单提交install_params.jsp

```
<form id="license_form" action="install_params.jsp" method="post">
```

（2）安装参数设置

设置安装参数（数据库信息，项目部署信息等），设置完成参数，提交表单，交由InstallServelt来处理。
处理过程：创建数据库、创建表、初始化数据、更新配置（更新jc_site和jc_config表信息）、处理数据库配置文件（/WEB-INF/config/jdbc.properties，完成数据库配置文件的字符串替换）、处理web.xml（FileUtils.copyFile方法完成web.xml的替换）。

注：文件位置的定位

```
getServletContext().getRealPath(webXmlFrom);
```

（3）最后成功跳转到安装完成页面

```
RequestDispatcher dispatcher = request
    .getRequestDispatcher("/install/install_setup.jsp");
dispatcher.forward(request, response);
```

注：安装过程主要使用Install类完成数据库操作，文件复制等操作。

（4）涉及到的文件处理类

```
String s = FileUtils.readFileToString(new File(fileName));//获取文件字符串
FileUtils.writeStringToFile(new File(fileName), s);//将字符串写入到文件
FileUtils.copyFile(new File(fromFile), new File(toFile));//拷贝文件
```

### 3. 源码安装

1）下载jeecms源码文件

下载软件地址：http://www.jeecms.com/

解压文件目录：db目录（mysql数据的数据文件）；src目录（java源码文件存放位置）；WebContent目录（存放web资源的文件夹）。

2）导入到eclipse中

（1）新建一个web项目--jeecms

（2）复制解压得到的文件到相应的web项目下

（3）配置文件位于：WEB-INF/config目录

（4）创建数据库，执行db文件夹下的建库语句

注：修改过数据库配置文件：jdbc.properties

（5）运行

部署到tomcat下运行，项目能够成功运行

3）运行问题

问题1：页面的样式和图像无法找到？

1）原因：http://localhost:8080/r/cms/www/default/css/html5.css ，在访问资源时没有响应的项目路径（即部署到tomcat的项目名）。

2）访问http://localhost:8080/jeecms/

访问的资源定位为：/WEB-INF/t/cms/www/default/index/index.html

注：具体过程查看DynamicPageAct类

3）controller层方法处理

查看com.jeecms.cms.action.front.DynamicPageAct类，该类是根路径访问的入口类。

该类的index方法实现首页的显示：

```
@RequestMapping(value = "/", method = RequestMethod.GET)
```

4）返回页面

访问到首页文件：/WEB-INF/t/cms/www/default/index/index.html

5）解决方法

admin登录后台，配置--全局设置--部署路径中输入/jeecms

注：jeecms是在tomcat下部署的项目的外层文件夹。

### 4. 后台访问以及用户密码修改

1）后台访问地址：http://localhost:8080/jeecms/jeeadmin/jeecms/

2）密码修改：

在Md5PwdEncoder类中新建一个main方法，生成一个密码，替换jo_user表的password字段值(update语句修改)。

```
public static void main(String[] args) {
    Md5PwdEncoder md5=new Md5PwdEncoder();
    System.out.println(md5.encodePassword("admin"));
}
```

### 5. 用户登录验证过程

1）模型和表

用户登录表为：jo_user，相应的hibernate配置文件为：UnifiedUser.hbm.xml

注：映射文件目录（src/com/jeecms/core/entity/hbm/），对应的实体类（UnifiedUser.java）

2）用户信息controller层类，services层类和dao层类

控制层：UnifiedUserAct.java

services层：（接口）UnifiedUserMng.java（manager的缩写），（实现类）UnifiedUserMngImpl.java

dao层：（接口）UnifiedUserDao.java，（实现类）UnifiedUserDaoImpl.java

3）后台登录操作

后台地址连接：http://localhost:8080/jeecms/jeeadmin/jeecms/

（1）CmsLoginAct类作为controller类。

（2）处理用户操作认证的实现类：CmsAuthenticationFilter

（3）相关配置

web.xml中配置了contextConfigLocation位置为：/WEB-INF/config/shiro-context.xml

在该配置文件中指定了过滤路径

shiro-context.xml文件

```
<!-- Shiro Filter -->   
<bean id="adminUrlBean" class="com.jeecms.core.security.CmsAdminUrl">
    <property name="adminLogin" value="/jeeadmin/jeecms/login.do"/>
    <property name="adminPrefix" value="/jeeadmin/jeecms/"/>
</bean>
<bean id="authcFilter" class="com.jeecms.core.security.CmsAuthenticationFilter" 
parent="adminUrlBean">
    <property name="adminIndex" value="/jeeadmin/jeecms/index.do"/>
</bean>
```

web.xml中的配置

```
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
      <param-name>targetFilterLifecycle</param-name>
      <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

注1：DelegatingFilterProxy利用spring容器创建的作为一个代理的filter过滤器

注2：shiro是一个安全框架，用来实现系统的安全应用

4）认证用户类

CmsAuthorizingRealm类，配置在shiro-context.xml文件中

```
<bean id="authorizingRealm" class="com.jeecms.core.security.CmsAuthorizingRealm">
```

在该类中获取UnifiedUser对象并构造SimpleAuthenticationInfo对象完成认证

```
//认证过程
protected AuthenticationInfo doGetAuthenticationInfo(
        AuthenticationToken authcToken) throws AuthenticationException {
    UsernamePasswordToken token = (UsernamePasswordToken) authcToken;
    CmsUser user = cmsUserMng.findByUsername(token.getUsername());
    if (user != null) {
        UnifiedUser unifiedUser = unifiedUserMng.findById(user.getId());
        return new SimpleAuthenticationInfo(user.getUsername(), unifiedUser.getPassword(), getName());
    } else {
         return null;
    }
}
```

5）Apache Shiro

参考资料：http://jinnianshilongnian.iteye.com/blog/2018936

### 6. 会员登录操作

1）访问链接：http://localhost:8080/jeecms/login.jspx?

第一步：LoginAct.java类的submit方法接收用户的登录请求

```
@RequestMapping(value = "/login.jspx", method = RequestMethod.POST)
```

第二步：方法内调用AuthenticationMng.java类的login方法。

第三步：在login方法中调用UnifiedUserMng.java类的login方法，该方法用于从数据库中获取UnifiedUser对象。

注：在UnifiedUserMng.java中login方法通过传递用户名从数据库加载到UnifiedUser对象，然后判断密码是否正确。验证成功后返回UnifideUser对象。

第四步：AuthenticationMng.java类的login方法获取UnifiedUser对象，封装成Authentication认证对象，放置到session中。

```
session.setAttribute(request, response, AUTH_KEY, auth.getId());
```

2）登录实现类AuthenticationMngImpl

```
public Authentication login(String username, String password, String ip,
        HttpServletRequest request, HttpServletResponse response,
        SessionProvider session) throws UsernameNotFoundException,
        BadCredentialsException {
    UnifiedUser user = unifiedUserMng.login(username, password, ip);
    Authentication auth = new Authentication();
    auth.setUid(user.getId());
    auth.setUsername(user.getUsername());
    auth.setEmail(user.getEmail());
    auth.setLoginIp(ip);
    save(auth);
    session.setAttribute(request, response, AUTH_KEY, auth.getId());
    return auth;
}
```

