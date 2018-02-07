---
title: 开源的门户网站jeecms--架构分析
tags: [jeecms]
---

类的封装

### 1. dao层

1）dao层封装--继承体系（以UnifiedUserDao为例）

（1）定义接口UnifiedUserDao.java

注：接口中提供定义该模块需要实现的接口（在详设中定义出规范），该接口被上层的service调用。

（2）定义具体的实现类UnifiedUserDaoImpl.java

```
@Repository
public class UnifiedUserDaoImpl extends HibernateBaseDao<UnifiedUser, Integer>
        implements UnifiedUserDao
```

提供主键@Repository供spring来管理bean对象

继承HibernateBaseDao<UnifiedUser, Integer>，传递泛型对象为该模块的模型类

（3）实现protected Class<UnifiedUser> getEntityClass()方法

注：提供数据库查询时需要的模型class对象

```
entity = (T) getSession().get(getEntityClass(), id);

Criteria criteria = getSession().createCriteria(getEntityClass());
```

（4）实现UnifiedUserDao接口定义的方法

（5）提供Criteria的查询方式

2）HibernateBaseDao类封装底层的查询

（1）理解泛型

提供dao层公共的数据操作实现类，并利用泛型变量使返回对象适合更多的模块来使用。

```
public abstract class HibernateBaseDao<T, ID extends Serializable> extends
        HibernateSimpleDao {
    protected T get(ID id, boolean lock) {
        T entity;
        if (lock) {
            entity = (T) getSession().get(getEntityClass(), id,
                    LockMode.UPGRADE);
        } else {
            entity = (T) getSession().get(getEntityClass(), id);
        }
        return entity;
    }
...
}
```

（2）提供一个抽象方法，供子类实现，目的是获取当前模块的Entity Class对象

```
abstract protected Class<T> getEntityClass();//获取具体模块dao对应的实体类
```

3）HibernateSimpleDao类

该类引用sessionFactory，创建操作数据库的session对象，并提供hibernate底层的操作接口，实现查询。

```
protected Session getSession() {
    return sessionFactory.getCurrentSession();
}
```

注：该类引用sessionFactory，创建操作数据库的session对象。该类同时提供了各种公共的查询方法。

注2：为HibernateBaseDao提供了更底层的数据操作环境

4）配置

在application-context.xml文件中设置sessionFactory

```
<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingLocations" value="#{propertyUtils.getList('hibernate.hbm')}"/>   
    <property name="hibernateProperties">
        <value>
        hibernate.dialect=${hibernate.dialect}
        hibernate.show_sql=false
        hibernate.format_sql=false
        hibernate.query.substitutions=true 1, false 0
        hibernate.jdbc.batch_size=20
        hibernate.jdbc.use_streams_for_binary=true
        hibernate.temp.use_jdbc_metadata_defaults=false
        <!-- 是否自动提交事务 -->
        hibernate.connection.autocommit=true
        <!-- 指定hibernate在何时释放JDBC连接 -->
        hibernate.connection.release_mode=auto
            
        hibernate.cache.use_query_cache=true
        hibernate.cache.use_second_level_cache=true
        </value>
    </property>
    <!--配置二级缓存-->
    <property name="cacheRegionFactory">
        <ref local="cacheRegionFactory"/>
    </property>
    <!--拦截器-->
    <property name="entityInterceptor">
        <ref local="treeInterceptor"/>
    </property>
</bean>
```

注：treeInterceptor实现对channel（栏目树）进行修改时拦截处理。

注2：拦截器一般可用于实现实体类增删改的日志记录。

### 2. controller层

1）controller层封装--以UnifiedUserAct.java为例

查看会员列表

```
@RequiresPermissions("unified_user:v_list")
@RequestMapping("/unified_user/v_list.do")
public String list(Integer pageNo, HttpServletRequest request,ModelMap model) {
    Pagination pagination = manager.getPage(cpn(pageNo), CookieUtils.getPageSize(request));
    model.addAttribute("pagination", pagination);
    return "unified_user/list";
}
```

注1：@RequiresPermissions是由apache-shiro提供的注解，该注解要求当前Subject在执行被注解的方法时具备一个或多个对应的权限（即权限校验的注解）。

注2：ModelMap主要用于传递控制方法处理数据到结果页面对象（将controller层数据传递到jsp页面）。作用类似于request对象的setAttribute方法的作用，用来在一个请求过程中传递处理的数据。在页面上获取数据可以通过el表达式来实现。与ModelAndView的区别是，ModelAndView可以设置页面跳转的路径或视图。

注3：页面跳转，return跳转的文件路径字符串（return "unified_user/list";）

2）如何跳转

web.xml中配置（后台管理的mvc配置）

```
<servlet>
    <servlet-name>JeeCmsAdmin</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/config/jeecms-servlet-admin.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

注：确定前端控制器类所在的DispatcherServlet（即spring用于转发请求的控制器）。

jeecms-servlet-admin.xml文件

```
<!--包含action-->
<import resource="jeecms/jeecms-servlet-admin-action.xml"/>
```

查看该文件的freemarkerViewResolver配置。

```
<bean id="freemarkerViewResolver" class="com.jeecms.common.web.springmvc.RichFreeMarkerViewResolver">
    <property name="prefix" value="/jeecms_sys/"/>
    <property name="suffix" value=".html"/>
    <property name="contentType" value="text/html; charset=UTF-8"/>
    <property name="exposeRequestAttributes" value="false"/>
    <property name="exposeSessionAttributes" value="false"/>
    <property name="exposeSpringMacroHelpers" value="true"/>
</bean>
```

因此，实际跳转的路径文：/jeecms_sys/unified_user/list.html文件。与freemarkerConfig配置的templateLoaderPath进行组合产生服务器文件路径

后台服务器的位置：/WEB-INF/jeecms_sys/unified_user/list.html。

该文件在返回浏览器前，使用freemarker进行渲染操作。

3）页面freemarker渲染操作配置（freemarker引擎）

```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF"/>
    <!-- 定义freemarker变量 -->
    <property name="freemarkerVariables">
        <map>
            <!--在FCK编辑器中需要用到appBase，以确定connector路径。-->
            <entry key="appBase" value="/jeeadmin/jeecms"/>
            <!--后台管理权限控制-->
            <entry key="cms_perm" value-ref="cms_perm"/>
            <entry key="text_cut" value-ref="text_cut"/>
            <entry key="html_cut" value-ref="html_cut"/>
            <entry key="cms_content_list" value-ref="cms_content_list"/>
            <entry key="cms_content_page" value-ref="cms_content_page"/>
        </map>
    </property>
    <!-- freemarker参数设置 -->
    <property name="freemarkerSettings">
        <props>
            <prop key="template_update_delay">0</prop>
            <prop key="defaultEncoding">UTF-8</prop>
            <prop key="url_escaping_charset">UTF-8</prop>
            <prop key="locale">zh_CN</prop>
            <prop key="boolean_format">true,false</prop>
            <prop key="datetime_format">yyyy-MM-dd HH:mm:ss</prop>
            <prop key="date_format">yyyy-MM-dd</prop>
            <prop key="time_format">HH:mm:ss</prop>
            <prop key="number_format">0.######</prop>
            <prop key="whitespace_stripping">true</prop>
            <prop key="auto_import">/ftl/jeecms/index.ftl as p,/ftl/spring.ftl as s</prop>
        </props>
    </property>
</bean>
```

4）list.html页面的渲染

该页面是使用ftl语法编写的，因此在服务器端会使用配置好的freemarkerConfig进行页面的渲染，生成并输出html页面。

```
<prop key="auto_import">/ftl/jeecms/index.ftl as p,/ftl/spring.ftl as s</prop>
```

因此index.ftl和spring.ftl文件是会自动导入的，因此文件中定义的ftl命令是可以在list.html文件中直接使用的。

### 3. service 层

1）UnifiedUserMngImpl类

```
@Service
@Transactional
public class UnifiedUserMngImpl implements UnifiedUserMng

    @Transactional(readOnly = true)
    public UnifiedUser findById(Integer id) {
        UnifiedUser entity = dao.findById(id);
        return entity;
    }
```

注：通过注解@Service交由spring容器来处理对象的生命周期，@Transactional设置事务开启和关闭，因为注解配置在类上，会对于类中所有的方法起作用。针对find，get这种只读事务的方法需要单独在方法上添加

注2：spring的注解方式不会对父类中的方法进行事务，类似于aop不能拦截父类中定义的方法。

2）事务配置

```
<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
</bean>
<context:annotation-config/>
<tx:annotation-driven transaction-manager="transactionManager" />
```
