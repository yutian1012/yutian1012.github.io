---
title: 相关文档
tags: [javaEE]
---

1、相关框架图

https://www.cnblogs.com/littleCode/p/3926791.html

1）spring框架介绍

核心容器：核心容器提供Spring 框架的基本功能。核心容器的主要组件是BeanFactory ，它是工厂模式的实现。BeanFactory 使用控制反转（IOC ） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。
Spring 上下文：Spring 上下文是一个配置文件，向Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如JNDI 、EJB、电子邮件、国际化、校验和调度功能。
Spring AOP ： 通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了Spring 框架中。所以，可以很容易地使Spring 框架管理的任何对象支持AOP 。Spring AOP 模块为基于Spring 的应用程序中的对象提供了事务管理服务。通过使用Spring AOP ，不用依赖EJB 组件，就可以将声明性事务管理集成到应用程序中。
Spring DAO ：JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向JDBC 的异常遵从通用的DAO 异常层次结构。
Spring ORM ：Spring 框架插入了若干个ORM 框架，从而提供了ORM 的对象关系工具，其中包括JDO 、Hibernate 和iBatisSQLMap 。所有这些都遵从Spring 的通用事务和DAO 异常层次结构。

2）整合mybatis

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="configLocation" value="classpath:/conf/configuration.xml"/>
    <property name="dataSource" ref="dataSource" />
    <property name="mapperLocations" >
        <list>
            <value>classpath:/com/hotent/*/maper/*.map.xml</value>
            <value>classpath:/com/haiya/*/maper/*.map.xml</value>
            <!-- 专利接口扫描文件 -->
            <value>classpath:/org/ipph/patent/net/zhihui/map/*.map.xml</value>
            <value>classpath:/org/ipph/patent/net/cpquery/map/*.map.xml</value>
        </list>
    </property>
</bean>

<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

mybatis的核心类，以及spring如何包装mybatis，对外提供统一的功能？？

3）j2EE框架

J2EE 核心是一组技术规范与指南，其中所包含的各类组件、服务架构及技术层次，均有共通的标准及规格，让各种依循J2EE 架构的不同平台之间，存在良好的兼容性，解决过去企业后端使用的信息产品彼此之间无法兼容，导致企业内部或外部难以互通的窘境。

4）框架拆分

参考：https://www.cnblogs.com/doudouxiaoye/p/5700669.html

**Bean组件**

Bean组件在Spring的org.springframework.beans包下，这个包下的所有类主要解决了三件事：Bean的定义、Bean的创建以及对Bean的解析。对Spring的使用者来说唯一需要关心的就是Bean的创建，其他两个由Spring在内部帮你完成了，对你来说是透明的。

1. Bean 工厂的继承关系

BeanFactory工厂类接口，最终的默认实现类是 DefaultListableBeanFactory。

2. Bean 定义的类层次关系图

在Spring的内部他就被转化成BeanDefinition对象（通过解析xml中的bean标签实现）。

3.  Bean 的解析类

BeanDefinitionReader和BeanDefinitionDocumentReader接口

**Context 组件**

context 在 Spring 的 org.springframework.context 包下，前面已经讲解了 Context 组件在Spring中的作用，他实际上就是给Spring提供一个运行时的环境，用以保存各个对象的状态。

1. ApplicationContext

ApplicationContext 是 Context 的顶级父类

ApplicationContext 必须要完成以下几件事：

```
标识一个应用环境
利用 BeanFactory 创建 Bean 对象
保存对象关系表
能够捕获各种事件
```

**Core 组件**

Core 组件作为 Spring 的核心组件，他其中包含了很多的关键类，其中一个重要组成部分就是定义了资源的访问方式。这种把所有资源都抽象成一个接口的方式很值得在以后的设计中拿来学习。

1. core组件中的Resource资源处理类

Resource 接口封装了各种可能的资源类型，也就是对使用者来说屏蔽了文件类型的不同。对资源的提供者来说，如何把资源包装起来交给其他人用这也是一个问题，我们看到 Resource 接口继承了 InputStreamSource 接口，这个接口中有个 getInputStream 方法，返回的是 InputStream 类。这样所有的资源都被可以通过 InputStream 这个类来获取，所以也屏蔽了资源的提供者。另外还有一个问题就是加载资源的问题，也就是资源的加载者要统一，从上图中可以看出这个任务是由 ResourceLoader 接口完成，他屏蔽了所有的资源加载者的差异，只需要实现这个接口就可以加载所有的资源，他的默认实现是 DefaultResourceLoader。

2. Context 和 Resource 的类关系

**Ioc 容器如何工作**

1）如何创建BeanFactory 工厂

Ioc 容器实际上就是Context组件结合其他两个组件共同构建了一个Bean关系网，如何构建这个关系网？构建的入口就在 AbstractApplicationContext 类的 refresh 方法中

refresh是一个核心方法

```
构建 BeanFactory，以便于产生所需的“演员”
注册可能感兴趣的事件
创建 Bean 实例对象
触发被监听的事件
```

2）如何加载解析bean呢？

refreshBeanFactory 方法中有一行loadBeanDefinitions(beanFactory) 将找到答案，该方法位于AbstractRefreshableApplicationContext类中。内部又会调用XmlBeanDefinitionReader方法。

注：存放到beanDefinitionMap集合中

3）向容器中添加一些工具类

创建好 BeanFactory 后，接下去添加一些 Spring 本身需要的一些工具类，这个操作在 AbstractApplicationContext 的 prepareBeanFactory 方法完成。

4）spring提供的扩展入口

refresh方法的第4，5，6句语句实现的功能，前两行主要是让你现在可以对已经构建的 BeanFactory的配置做修改，后面一行就是让你可以对以后再创建Bean的实例对象时添加一些自定义的操作

```
// Allows post-processing of the bean factory in context subclasses. 
postProcessBeanFactory(beanFactory); 
// Invoke factory processors registered as beans in the context. 
//在 invokeBeanFactoryPostProcessors 方法中主要是获取实现 BeanFactoryPostProcessor 接口的子类。并执行它的 postProcessBeanFactory 方法
invokeBeanFactoryPostProcessors(beanFactory); 
// Register bean processors that intercept bean creation. 
registerBeanPostProcessors(beanFactory); 
```

注：可结合spring bean的生命周期图查看

查看2018-08-05-spring-bean-initial-class文件。

5）事件监听

refresh方法的下面几行代码

```
 // Initialize event multicaster for this context. 
initApplicationEventMulticaster(); 
...
// Check for listener beans and register them. 
registerListeners(); 
```

后面的几行代码是初始化监听事件和对系统的其他监听者的注册，监听者必须是 ApplicationListener 的子类。

6）创建 Bean 实例并构建 Bean 的关系网

上面的操作主要是准备环境上下文，已经加载解析xml等定义的beanDefinition对象，并没有进行bean对象的生成。

Bean 的实例化代码，是从 finishBeanFactoryInitialization 方法开始的，Bean 的实例化是在 BeanFactory 中发生的。preInstantiateSingletons 方法。

注：DefaultListableBeanFactory.preInstantiateSingletons的具体实现

7）创建bean实例时有一个关键类FactoryBean

可以说 Spring 一大半的扩展的功能都与这个 Bean 有关，这是个特殊的Bean他是个工厂Bean（是可以产生Bean的Bean，即该对象可以产生相关的bean对象，通过实现FactoryBean的接口getObject方法）。

如果beanDefinition中定义的是bean是FactoryBean，则实例化bean对象时，要调用getObject方法获取实例对象。

8）bean的关系组织情况

重点代码位于AbstractAutowireCapableBeanFactory类中。

9）一个扩展点的使用

给groovy脚本引擎注入 service对象引用和实现IScript对象的引用。

查看类GroovyScriptEngine，该类实现了BeanPostProcessor

```
public Object postProcessAfterInitialization(Object bean, String beanName)
        throws BeansException {
    boolean rtn = BeanUtils.isInherit(bean.getClass(), BaseService.class);
    boolean isImplScript = BeanUtils.isInherit(bean.getClass(),IScript.class);
    if (rtn || isImplScript) {
        binding.setProperty(beanName, bean);
    }
    return bean;
}
```

注：完成向GroovyBinding中注入属性信息。GroovyBinding继承了Binding（groovy.lang.Binding）。该类中Property使用map集合存储，执行时的参数使用ThreadLocal处理，在ThreadLocal使用完成后最好手动清除该对象关联的引用。

引擎的执行

```
GroovyShell shell = new GroovyShell(binding);
setParameters(shell,vars);

//待运行的脚本
script=script.replace("&apos;", "'")
.replace("&quot;", "\"")
.replace("&gt;", ">")
.replace("&lt;", "<")
.replace("&nuot;", "\n")
.replace("&amp;", "&");

Object rtn =  shell.evaluate(script);

return rtn;
```

**Ioc 容器的扩展点**

BeanFactoryPostProcessor， BeanPostProcessor。他们分别是在构建 BeanFactory 和构建 Bean 对象时调用。还有就是 InitializingBean 和 DisposableBean 他们分别是在 Bean 实例创建和销毁时被调用。用户可以实现这些接口中定义的方法，Spring 就会在适当的时候调用他们。还有一个是 FactoryBean 他是个特殊的 Bean，这个 Bean 可以被用户更多的控制。

注：容器的获取可以通过实现ApplicationContextAware接口，会自动注入到相应的bean中

**spring AOP**

就必须先了解的动态代理的原理，因为 AOP 就是基于动态代理实现的。

1）动态代理相关类

Proxy，InvocationHandler，以及被代理的接口（可多个）。底层实现会涉及到ProxyGenerator 类在 sun.misc 包下。

2）spring配置代理

代理的目的是调用目标方法时我们可以转而执行 InvocationHandler 类的 invoke 方法，所以如何在 InvocationHandler 上做文章就是 Spring 实现 Aop 的关键所在。

注：Spring 引用了 Aop Alliance定义的接口，底层会使用到JdkDynamicAopProxy对象（位于spring-aop包中）。在这里 JdkDynamicAopProxy 类实现了 InvocationHandler 接口。

1. 配置方式代理：

配置上看到要设置被代理的接口，和接口的实现类也就是目标类，以及拦截器也就在执行目标方法之前被调用，这里 Spring 中定义的各种各样的拦截器，可以选择使用

```
<bean id="testBeanSingleton" 
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces">
        <value>
            org.springframework.aop.framework.PrototypeTargetTests$TestBean
        </value>
    </property>
    <property name="target"><ref local="testBeanTarget"></ref> </property>
    <property name="singleton"><value>true</value></property>
    <property name="interceptorNames">
        <list>
            <value>testInterceptor</value>
            <value>testInterceptor2</value>
        </list>
    </property>
</bean>
```

注：使用ProxyFactoryBean可获取代理对象，因为这是一个FactoryBean，获取对象可通过getObject方法，因此逻辑都被定义在了该方法中。

注2：通过ProxyFactoryBean我们也可以学习到如何扩展自己的代码，创建出相应的代理，或更具有奇思妙想的对象来。

原理：

JdkDynamicAopProxy中会获取配置在目标对象的上的拦截器链，调用ReflectiveMethodInvocation，创建根据执行的方法运行相应的拦截器。

spring aop代理可对目标对象配置相应的拦截器，执行拦截器的前置和后置方法，最后调用目标对象的目标方法，从而完成整个业务过程。

2. 相关概念

切面：

利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

关注点：

核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志，事务。

1、横切关注点

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2、切面（aspect）--类，包含相关业务处理方法

类是对物体特征的抽象，切面就是对横切关注点的抽象

3、连接点（joinpoint）

被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

注：目标对象中被拦截的方法

4、切入点（pointcut）--定义连接点表达式

对连接点进行拦截的定义

5、通知（advice）--拦截后的处理时机

所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

6、目标对象--待拦截的对象

代理的目标对象

7、织入（weave）

将切面应用到目标对象并导致代理对象创建的过程

8、引入（introduction）

在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段

实例配置

aop:config标签，aop:pointcut标签，aop:advisor标签，aop:aspect标签，aop:before标签，aop:after-returning标签，aop:around

Advisor是切面的另一种实现，能够将通知以更为复杂的方式织入到目标对象中，是将通知包装为更复杂切面的装配器。Advisor由切入点和Advice组成。

注：Adivisor是一种特殊的Aspect，Advisor代表spring中的Aspect

注2：事务配置通过使用Adivisor实现，其他类型的aop配置可通过aop:aspect实现

```
<!-- 配置事务使用adivisor -->
<aop:config proxy-target-class="true">
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.hotent.platform.service..*.*(..))"  />
</aop:config>
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="save_newTransaction" propagation="REQUIRES_NEW"/>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="is*" read-only="true"/>
        <tx:method name="find*" read-only="true"/>
        <tx:method name="*" isolation="READ_COMMITTED" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>

<!-- 配置其他aop，使用aop:aspect方式 -->
<aop:config proxy-target-class="true">
    <aop:aspect ref="logAspect" >
        <!-- 定义切点，以及在该切点出执行的方法 -->
        <aop:pointcut expression="execution(* com.hotent.platform.controller..*.*(..))" id="logPointcut"/>
        <aop:around pointcut-ref="logPointcut" method="doAudit"/>
        <aop:pointcut expression="execution(* com.hotent.ipph.controller..*.*(..))" id="logPointcutIpph"/>
        <aop:around pointcut-ref="logPointcutIpph" method="doAudit"/>
    </aop:aspect>
</aop:config>
```

注意区分隔离级别和传播级别，传播级别是事务与事务之间的关系，而隔离级别涉及到数据的隔离级别的支持。mysql默认是可重复读，上面设置的隔离级别为读已提交（可能出现幻读问题）。

扩展：利用aop实现系统数据权限的过滤和认证

系统中通过创建相关的权限注解，实现权限拦截。权限内容设置以map的形式存储，并通过拦截方法提供的map参数，将数据设置到map中（代码有侵入性）。

注：spring aop无法拦截父类方法，通过service类的多个事务方法调用时，也不能有效。这里强调的是同一个类的多个方法，而不是使用spring相应注解注入的对象。

扩展2：一些页面样式模板类使用Freemarker引擎，定义成相应的ftl模板。这部分内容包括消息固定样式等信息，邮件发送模板。

**spring中的设计模式**



**spring mvc**

**spring security**

**项目中用到的设计模式**

建造器类，模板方法，策略模式，代理模式