---
title: 自定义异常类的使用
tags: [work]
---

eip系统对外提供webservice服务，开放服务供外部调用。

1）系统配置信息

app-context.xml中配置信息

```
<import resource="app-cxf-service.xml"/>
```

web.xml中配置

```
<servlet>
    <servlet-name>cxf</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <!-- 项目启动的时候加载 -->
    <load-on-startup>1</load-on-startup>
</servlet>
```

注：与相关的ws配置结合，构造出ws的访问地址。

app-cxf-service.xml文件配置

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jaxws="http://cxf.apache.org/jaxws"
    xmlns:jaxrs="http://cxf.apache.org/jaxrs"
    xmlns:cxf="http://cxf.apache.org/core"
    xsi:schemaLocation="http://cxf.apache.org/core 
                        http://cxf.apache.org/schemas/core.xsd 
                        http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://cxf.apache.org/jaxws 
                        http://cxf.apache.org/schemas/jaxws.xsd
                        http://cxf.apache.org/jaxrs    
                        http://cxf.apache.org/schemas/jaxrs.xsd
                        ">
    <!-- 引入cxf的jar包中提供的cxf配置信息，这些配置中定义了关键的bean对象
        方便在cxf命名空间中直接饮用相关类 -->
    <import resource="classpath:META-INF/cxf/cxf.xml" />
    <import resource="classpath:META-INF/cxf/cxf-extension-soap.xml"/>
    <import resource="classpath:META-INF/cxf/cxf-servlet.xml" />

    <!-- 配置的一个ws服务 -->
    <!-- 服务实现类，登陆用户-->
    <bean id="UserDetailsServiceImpl" class="com.hotent.platform.webservice.impl.UserDetailsServiceImpl"/>

    <!-- endpoint方式发布ws，endpoint也可以单独制定ws服务地址 -->
    <jaxws:endpoint    
        id="UserDetailsService"    
        implementor="#UserDetailsServiceImpl" 
        implementorClass="com.hotent.platform.webservice.impl.UserDetailsServiceImpl"   
        address="/UserDetailsService" >

        <!-- 配置拦截器 -->
        <jaxws:inInterceptors>  
            <ref bean="addressInInterceptor" />
        </jaxws:inInterceptors>
        <jaxws:outInterceptors>
            <ref bean="afterInterceptor" /> 
        </jaxws:outInterceptors>
    </jaxws:endpoint>
</beans>
```

2）UserDetailsService服务的接口和实现类

接口UserDetailsService类

```
import java.util.List;

import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebService;
import javax.jws.soap.SOAPBinding;
import javax.jws.soap.SOAPBinding.ParameterStyle;
import javax.jws.soap.SOAPBinding.Style;

import org.springframework.dao.DataAccessException;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import com.hotent.platform.model.system.SysOrg;
import com.hotent.platform.model.system.SysUser;

//SOAPBing指定ws使用的soap message协议
@SOAPBinding(style=Style.DOCUMENT,parameterStyle=ParameterStyle.WRAPPED)
//定义webservice服务
@WebService(endpointInterface="com.hotent.platform.webservice.api.UserDetailsService",
            targetNamespace="http://impl.webservice.platform.hotent.com/")
public interface UserDetailsService {
    
    /**
     * 取得适用于Spring-security的用户对像UserDetail。
     * @param userName  用户名
     * @return SysUser  适用于Spring-security的用户对像UserDetail。
     * @throws UsernameNotFoundException    当不存在该用户时会抛出此异常。
     * @throws DataAccessException  当连接数据库出错时会抛出此异常。
     */
    @WebMethod(operationName="loadUserByUsername")
    public SysUser loadUserByUsername(@WebParam(name="userAccount") String userName)throws Exception;
    
    ...
}
```

注：SOAPBinding，WebService，WebMethod等注解都是有jdk提供的。

实现类UserDetailsServiceImpl

```
public class UserDetailsServiceImpl implements UserDetailsService {
    @Resource
    SysUserDao sysUserDao;
    @Resource
    private SysRoleService sysRoleService;
    @Resource
    private SysOrgService sysOrgService;
    @Resource
    private SysPaurService sysPaurService;

    /**
     * 取得登陆子系统的user
     * 
     * @param userName
     * @return
     * @throws UsernameNotFoundException
     * @throws DataAccessException
     */
    public SysUser loadUserByUsername(String account) throws Exception {
        UserDetails user = sysUserDao.loadUserByUsername(account);
        if (user instanceof SysUser) {
            SysUser sysUser = (SysUser) user;
            return sysUser;
        } else
            return null;
    }
    ....
}
```

注：ws的注解也会被UserDetailsServiceImpl所继承