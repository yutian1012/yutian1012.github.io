---
title: freemarker环境
tags: [freemarker]
---

### 1. freemarker应用程序的使用

1）引入freemarker的jar包

```
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.23</version>
</dependency>
```

2）解析模板

创建类解析ftl模板文件(FreemarkerProcess)

```
public class FreemarkerProcess {
    public static void main(String[] args) {
        ////指定一个不兼容的版本号
        Configuration cfg = new Configuration(Configuration.DEFAULT_INCOMPATIBLE_IMPROVEMENTS);
        Template template = null;
        try {
            //使用FreemarkerProcess类的getResource方法加载test.ftl文件
            //cfg.setClassForTemplateLoading(FreemarkerProcess.class,"");
            //使用文档路径指定模板文件的模板
            //cfg.setDirectoryForTemplateLoading(new File("G:\\Workspaces\\springTest Maven Webapp\\src\\main\\java\\org\\ipph\\web\\ftl"));
            //利用classLoader来指定模板文件位置
            cfg.setClassLoaderForTemplateLoading(FreemarkerProcess.class.getClassLoader(), "org/ipph/web/ftl");
            //局部的变量，在process中传入
            Map<String,String> data=new HashMap<String,String>();
            data.put("hello", "helloworld");
            template=cfg.getTemplate("test.ftl");
            template.process(data, new OutputStreamWriter(System.out));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

创建模板文件(test.ftl)

```
${hello}
```

注：上面的程序中使用了3种方式来指定模板文件的位置，分别使用Class的getResource，File目录，ClassLoader的getResource方法来加载模板文件。

注2：template.process中传入的map集合是一个局部的数据集，全局的数据集可以在Configuration中设置

```
Map<String,String> data=new HashMap<String,String>();
data.put("hello", "helloworld");
cfg.setSharedVaribles(data);
template.process(null, new OutputStreamWriter(System.out));
```

### 2. freemarker web应用

结合spring mvc实现web页面渲染

1）导入依赖的jar包(在已配置好的spring mvc上增加jar)

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>${spring.version}</version>
</dependency>

<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.23</version>
</dependency>
```

注意：

spring-context-support一定要导入（spring-context-support.jar）这个jar文件包含支持UI模版（Velocity，FreeMarker，JasperReports），邮件服务，脚本服务(JRuby)，缓存Cache（EHCache），任务计划Scheduling（uartz）方面的类。

同时，使用时需要导入外部依赖spring-context, (spring-jdbc, Velocity, FreeMarker, JasperReports, BSH, Groovy, JRuby, Quartz, EHCache)。

2）配置页面处理器

```
<!-- Freemarker配置 -->
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">  
    <property name="templateLoaderPath" value="/WEB-INF" />
    <property name="freemarkerSettings">  
        <props>  
            <prop key="template_update_delay">0</prop>  
            <prop key="default_encoding">UTF-8</prop>  
            <prop key="number_format">0.##########</prop>  
            <prop key="datetime_format">yyyy-MM-dd HH:mm:ss</prop>  
            <prop key="classic_compatible">true</prop>  
            <prop key="template_exception_handler">ignore</prop>  
        </props>  
    </property>  
</bean>
    
<!--视图解释器 -->  
<bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">  
    <property name="prefix" value="/jsp" />
    <property name="suffix" value=".jsp" />
    <property name="contentType" value="text/html;charset=UTF-8"></property>
</bean>
```

注：templateLoaderPath与prefix整合后的路径为/WEB-INF/jsp

注2：FreeMarkerConfigurer可以配置到spring容器中（这里配置到了spring mvc容器中）

3）处理器类HelloworldController

```
@Controller
@RequestMapping("/ipph/hello")
public class HelloworldController {
    @RequestMapping("/test1")
    public ModelAndView helloworld(HttpServletRequest request,HttpServletResponse response){
        return new ModelAndView("/ipph/hello/Helloworld")
            .addObject("hello","helloworld");
    }
}
```

注：通过addObject方法将hello变量设置值为helloworld

4）页面Helloworld.jsp

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>My JSP 'Helloworld.jsp' starting page</title>
  </head>
  <body>
   ${hello} <br>
  </body>
</html>
```

5）访问

部署并运行，访问：http://localhost:8080/springTest/ipph/hello/test1

6）注意事项

配置FreeMarkerConfigurer时，必须指定templateLoaderPath，可以指定空。但是与viewResolver的prefix，已经controller返回的字符串结合必须能够造成模板文件路径。并且可以只在templateLoaderPath中设置，而不在prefix中设置（该属性可省略）。

```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">  
    <property name="templateLoaderPath" value="" />
...
</bean>

<bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">  
    <property name="prefix" value="/WEB-INF/jsp" />
    <property name="suffix" value=".jsp" />
    <property name="contentType" value="text/html;charset=UTF-8"></property>
</bean>
```

ftl解析的数据可以设置到ModelAndView

```
return new ModelAndView("/ipph/hello/Helloworld")
    .addObject("hello","helloworld");
```

另外，数据也可设置到ModelMap以及request.setAttribute和session.setAttribute中

测试代码：

```
@Controller
@RequestMapping("/ipph/hello")
public class HelloworldController {
    @RequestMapping("/test1")
    public ModelAndView helloworld(HttpServletRequest request,HttpServletResponse response,ModelMap map){
        //将freemarker数据设置到ModelMap中，供页面展示
        //map.put("hello","hellomodel");
        //将freemarker数据设置到request中，供页面展示
        //request.setAttribute("hello", "hellorequest");
        //将freemarker数据设置到session中，供页面展示
        request.getSession().setAttribute("hello", "hellosession");
        return new ModelAndView("/ipph/hello/Helloworld");
        //.addObject("hello","helloworld");
    }
}
```

### 3. Configuration对象

比较上面的两种方式处理ftl模板，可以发现一个共同的处理步骤。首先创建Configuration对象，然后指定模板位置，最后使用Template对象处理。springmvc隐藏了内部细节，执行解析过程都是相同的。

1）Configuration对象

Configuration 是一个存放应用级别（application level）公共配置信息，以及模版（Template）可使用的全局共享变量的一个对象。同时它还负责模版（Template）实例的创建以及缓存。

Configuration 对象可被Template 对象的方法使用，每一个模版实例都关联与一个Configuration 实例，它是通过Template 的构造函数被关联进去的，通常是你使用这个方法来Configuration.getTemplate 获得模版对象的。

2）共享变量

共享变量是那些定义给所有模版（Template）使用的变量。你可以通过configuration对象的setSharedVariable 方法来添加共享变量。

```
Configuration cfg = new Configuration(); 
cfg.setSharedVariable("wrap", new WrapDirective()); cfg.setSharedVariable("company", "Foo Inc.");
//或者
cfg.setSharedVaribles(data);//data是一个Map对象
```

3）配置参数

配置参数是那些可以影响FreeMarker运行行为的那些命名参数。例如locale,number_format。

配置参数存储在Configuration实例中，它可以被模版实例（Template）修改。例如，你在Configuration中设置了locale等于"en_US"，那么所有的模版对象都会使用，"en_US"除非你在单个模版实例中利用setLocale方法修改了默认配置。因此configuration设置的参数可以当作是默认参数，它可以被Template一级设置的参数覆盖，而它们两者设置的参数信息又可以被环境中设置的参数所覆盖（也就是模版文件指令设置的）如下： 

```
<#setting locale="en_US">
```

参数设置

```
cfg.setSetting(name, value);
template.setSetting(name, value);
```

注：都是接收字符串类型的值

4）加载模板

模版加载器是基于抽象路径（"index.ftl"或"products/catalog.ftl"）加载原始数据的那些对象，而究竟加载何种资源（目录中的文件数据还是数据库中的数据）取决于具体的加载器实现。

（1）内建的模版加载器

```
//指定了一个文件系统中的目录，FreeMarker 将会在此目录加载模版
void setDirectoryForTemplateLoading(File dir);
//使用class的getResource来加载模板
void setClassForTemplateLoading(Class cl, String prefix);
//把web 应用的上下文以及基路径（相对与WEN-INF的父路进来说）作为参数，该种方式的模版加载器将会从web 应用上下文种加载模版。
void setServletContextForTemplateLoading(Object servletContext, String path);
```

注：这三个方法都是在Configuration类中提供的。

（2）从多个位置加载模版

如果你想从多个位置加载模版的话，你可以分别创建与不同位置对应的单个模版加载器，然后把它们包裹到一个名叫MultiTemplateLoader模版加载器中，最终通过方法setTemplateLoader（TemplateLoader加载器）把其设置给Configuration对象，以下有一个从两个不同位置加载模版的例子：

```
FileTemplateLoader ftl1 = new FileTemplateLoader(new File("/tmp/templates"));
FileTemplateLoader ftl2 = new FileTemplateLoader(new File("/usr/data/templates"));
ClassTemplateLoader ctl = new ClassTemplateLoader(getClass(), "");
TemplateLoader[] loaders = new TemplateLoader[] { ftl1, ftl2, ctl };
MultiTemplateLoader mtl = new MultiTemplateLoader(loaders);
cfg.setTemplateLoader(mtl);
```

注：FreeMarker将会首先在路径/tmp/templates中搜索模版文件，如果没有找到那么回到路径/usr/data/templates中搜索，如果还没有找到，那么则会尝试用class-loader的方式加载。

（3）从其他资源中获取模版文件

如果在这些内建的模版加载器中没有一个符合你的要求，那么你可以自己定制一个模版加载器，只需要实现freemarker.cache.TemplateLoader接口就可以了，然后通过方法setTemplateLoader(TemplateLoader loader)把其传递给Configuration对象。 

（4） 缓存模版

FreeMarker缓存模版的意思就是，当你通过getTemplate方法获取一个模版的时候，FreeMarker不仅会返回一个Template对象，而且会缓存该对象，当你下一次以相同的路径请求模版的时候，它就会返回缓存中的模版对象。

如果你改变了模版文件，那么当你下一次获取模版的时候，FreeMarker会自动重新加载，重新解析模版。虽然如此，但是如果直接判断一个文件是否修改过是一个耗时的操作，那么FreeMarker 在Configuration 对象级别提供了一个配置参数"update delay"。该参数的意思是FreeMarker多长时间去判断一次模版的版本，默认设置是5秒钟，也就是每个5秒就会判断模版是否经过修改，如果你想实时的判断，那么设置该参数为0。

另外一点需要注意，并不是所有的加载器都支持这种判断方式，举例来说基于class-loader 的模版加载器就不会发现你修改过模版文件。

对于删除缓存中的模版FreeMarker是这么做的，你可以使用Configuration对象方法clearTemplateCache以手工的方式清楚缓存中的模版对象。

实际上缓存部分可以作为一个组件加入到FreeMarker中（也就是它可以使用第三方缓存方案）你可以通过设置cache_storage这个参数来实现。对大多数开发者来FreeMarker自带的freemarker.cache.MruCacheStorage实现已经足够了。

MruCacheStorage缓存使用2个级别的Most Recently Used（最近最多用）策略。在第一个级别，所有的缓存条目都是使用强引用（strongly referenced：条目并不会被JVM 所清除，与其相对的弱引用softly reference）直到达到最大时间，那些最近最少使用的条目就会被迁移到二级缓存。在这个级别条目都是使用弱引用直到达到过期。弱引用与强引用的区域的大小是可以在构造函数中设置的。

例如你想把强引用区域设置为20，弱引用区域设置为250，那你可以使用以下代码： 

```
cfg.setCacheStorage(new freemarker.cache.MruCacheStorage(20, 250))
```

由于MruCacheStorage 是默认的缓存实现，那么你也可以这样设置：

```
cfg.setSetting(Configuration.CACHE_STORAGE_KEY,"strong:20, soft:250");
```

注：当你创建一个新的Configuration时，其默认使用MruCacheStorage缓存实现且默认的值maxStrongSize等于0，maxSoftSize等于Integer.MAX_VALUE（也就是理论最大值）。但是对于高负荷的系统来说，我们建议maxStrongSize设置成一个非0的数值，不然会导致频繁的重新加载，重新解析模版。

5）异常处理

FreeMarker 产生的异常一般可归以下几类：

1）FreeMarker 初始化阶段产生的异常： 

通常在你的应用中仅需要初始化FreeMarker一次，而当在这个时间段内产生的异常就叫做初始化异常。

2）加载解析模版期的异常：

当你通过Configuration.getTemplate()方法获取模版的时候（如果模版之前没有被缓存），将会产生两类异常：

（1）IOException：

由于模版没有找到，或在读取模版的时候发生其他的IO异常，比如你没有读取该文件的权限等等；

（2）freemarker.core.ParseException：

由于模版文件的语法使用不正确；

3）执行期间的异常：

当你调用Template.process(...)方法的时候，会抛出两类异常：

（1）IOException：

往输出写数据时候发生的错误；

（2）freemarker.template.TemplatException：

其他运行期产生的异常，比如一个最常见的错误就是模版引用了一个不存在的变量。

参考：https://my.oschina.net/u/699015/blog/175704

