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

在获取权限集合时调用env.getDataModel().get(PERMISSION_MODEL)，获取的权限信息是在Environment类env变量的rootDataModel（TemplateHashModel类型）成员变量中获取的，即在execute方法调用前Environment变量实例化时被赋值的。

注：具体Environment实例化是在freemarker解析ftl模板时，遇到自定义表情时对Environment实例化的。

### 4. data_model

```
<#list .data_model?keys as pkey>
    <#if pkey?starts_with('query')>
<input type="hidden" name="${pkey}" value="${(.data_model[pkey])!?string}"/>
</#if><#t/>
</#list>
```
注：这里的.data_model是freemarker顶层的集合，即后台传递的ModelMap集合对象。

This works if the data-model is just a usual Map or JavaBean, but for more sophisticated（复杂的，先进的） data-models it's up to the data-model implementation if it supports ?keys and if it indeed returns everything.
You also have the variables that you set in the templates, which can be listed like above, only instead of .data_moder use .globals and .namespace (means the current template namespace) and .locals.

### 5. 自定义标签实例

cms_content_list标签的处理

1）TemplateDirectiveModel接口

该接口是freemarker自定标签或者自定义指令的核心处理接口

2）自定义类实现该接口的execute方法

```
public void execute(Environment env, Map params,
 TemplateModel[] loopVars,TemplateDirectiveBody body)
```

execute方法的参数

（1）@param env：系统环境变量

获取输出流：Writer out = env.getOut();

模板参数：freemarker引擎解释模板时使用的是env环境中的参数

（2）@param params：自定义标签传过来的对象，其key=自定义标签的参数名，value值是TemplateModel类型。

TemplateModel是一个接口类型，通常我们都使用TemplateScalarModel接口来替代它获取一个String 值，如TemplateScalarModel.getAsString();当然还有其它常用的替代接口，如TemplateNumberModel获取number，TemplateHashModel等。

（3）@param loopVars  循环替代变量

（4）@param body 用于处理自定义标签中的内容（标签体）

freemarker解析标签体显示内容：body.render(env.getOut());

4）自定义的实现

通过params获取查询参数，调用相应的service查询出数据

5）配置自定义标签

在jeecms-servlet-admin.xml中配置的freemarkerConfig的freemarkerVariables属性中添

```
<entry key="cms_content_list" value-ref="cms_content_list"/>
```

注：那么在页面上就可以调用自定义标签cms_content了。

6）内容的展现

```
<div id="tabs-1">
    <table width="100%" border="0" cellspacing="0" cellpadding="0">
        <@cms_content_list count='10' titLen='12' orderBy='8' channelOption='1' >
           <#list tag_list as a>
             <tr>
                <td width="85%" height="27" align="left" style="background-color:#dceef4; 
border-bottom:1px solid #ffffff; text-indent:1.5em;">
                    <a href="${a.url}" title="${a.title}" target="_blank">
                       <span><@text_cut s=a.title len=titLen /></span>
                    </a>
                </td>
                <td style="background-color:#dceef4; border-bottom:1px solid #ffffff;">
                    <em>${a.viewsMonth!}</em>
                </td>
              </tr>
            </#list>
        </@cms_content_list>
</table>  
</div>
```

注：在自定义标签<@cms_content_list>中使用了<#list tag_list as a>，list遍历集合tag_list。

集合tag_list是如何获取的呢？查看ContentListDirective类的execute方法

```
List<Content> list = getList(params, env);
Map<String, TemplateModel> paramWrap = new HashMap<String, TemplateModel>(params);
//这里OUT_LIST的值为tag_list，即指令body体中遍历的集合
paramWrap.put(OUT_LIST, DEFAULT_WRAPPER.wrap(list));
//env环境中的元素数据，而将env环境值设置为当前传递的参数
Map<String, TemplateModel> origMap = DirectiveUtils.addParamsToVariable(env, paramWrap);
....
//使用标签体来显示list内容
//body变量为execute方法的参数TemplateDirectiveBody body
body.render(env.getOut());
```
