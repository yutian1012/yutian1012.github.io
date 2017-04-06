---
title: 开源的门户网站jeecms--页面ftl语法
tags: [jeecms]
---

以文件/WEB-INF/jeecms_sys/unified_user/list.html为例

### 1. list.html页面使用到的指令

1）<#include "/jeecms_sys/head.html"/>指令，引入head.html页面

2）指令

```
<@s.m "global.position"/>
```

```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    ...
    <property name="freemarkerSettings">
        <props>
            ...
            <prop key="auto_import">/ftl/jeecms/index.ftl as p,/ftl/spring.ftl as s</prop>
        </props>
    </property>
</bean>
```

注：自动引入index.ftl和spring.ftl模板库，在该文件中使用macro定义了很多ftl指令。位于spring.ftl模板文件中，实现国际化的指令。

3）获取值

```
<input type="hidden" name="pageNo" value="${pageNo!}"/>
//${pageNo!}从顶层变量中获取值，并设置默认值为空（符号!）
```

注：UnifiedUserAct类的list方法中设置的model.addAttribute("pagination", pagination);

4）table标签

```
<@p.table value=pagination;user,i,has_next><#rt/>
//这一行使用了两条命令：<@p.table>和<#rt/>
```

table命令通过/ftl/jeecms/index.ftl引入，实际定义在/WEB-INF/ftl/jeecms/ui/table.ftl文件中。

注：该命令主要是遍历查询到的集合对象，生成表格显示到页面，同时生成分页控件。

6）页面显示操作提醒

```
<#include "/common/alert_message.html"/>
```

引入提醒信息，alert_message.html内容

```
<#if message??>
<script type="text/javascript">
$.alert("<@s.m 'global.prompt'/>","<@s.mt code=message text=message/>");
</script>
</#if>
```

### 2. ftl命令定义文件

以spring.ftl文件为例

```
<#ftl strip_whitespace=true>
```

使用ftl 指令在模板中开始或者关闭处理空白的功能。

```
<#macro m code>${springMacroRequestContext.getMessage(code)}</#macro>
```

AbstractTemplateView 这个类里面有 

```
public static final String SPRING_MACRO_REQUEST_CONTEXT_ATTRIBUTE
 = "springMacroRequestContext"; 
model.put(SPRING_MACRO_REQUEST_CONTEXT_ATTRIBUTE, 
new RequestContext(request, model));
```

注：使用的对象是RequestContext（org.springframework.web.servlet.support.RequestContext）。getMessage方法是用来实现国际化的方法。

### 3. ftl自定义标签

1）cms_perm标签（后台权限标签）

定义位置：jeecms-context.xml

```
<bean id="cms_perm" class="com.jeecms.cms.web.PermistionDirective"/>
```

注：主要是根据标签中设置的url判断当前用户能否访问

freemarker解析引擎中注入了该权限标签

```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF"/>
    <property name="freemarkerVariables">
        <map>
            ...
            <entry key="cms_perm" value-ref="cms_perm"/>
        </map>
    </property>
    ...
</bean>
```

2）标签实现

```
public class PermistionDirective implements TemplateDirectiveModel {
```

实现TemplateDirectiveModel接口

问题：权限集合是如何获取？

```
private TemplateSequenceModel getPerms(Environment env)
        throws TemplateModelException {
    TemplateModel model = env.getDataModel().get(PERMISSION_MODEL);
    if (model == null) {
        return null;
    }
    if (model instanceof TemplateSequenceModel) {
        return (TemplateSequenceModel) model;
    } else {
        throw new TemplateModelException(
            "'perms' in data model not a TemplateSequenceModel");
    }
}
//env.getDataModel().get(PERMISSION_MODEL);获取的对象是什么时候设置的？
```

AdminContextInterceptor类的postHandle方法设置权限到ModelMap中

```
@Override
public void postHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler, ModelAndView mav)
        throws Exception {
    CmsUser user = CmsUtils.getUser(request);
    CmsSite site=CmsUtils.getSite(request);
    // 不控制权限时perm为null，PermistionDirective标签将以此作为依据不处理权限问题。
    if (auth && user != null && !user.isSuper() && mav != null
        && mav.getModelMap() != null && mav.getViewName() != null
        && !mav.getViewName().startsWith("redirect:")) {
        mav.getModelMap().addAttribute(PERMISSION_MODEL, getUserPermission(site, user));
    }
    if (mav != null&& mav.getModelMap() != null) {
        mav.getModelMap().addAttribute(SSO_ENABLE,CmsUtils.getSite(request).getConfig().getSsoEnable());
    }
}
//freemarker为什么能获取ModelMap中的值？
```

