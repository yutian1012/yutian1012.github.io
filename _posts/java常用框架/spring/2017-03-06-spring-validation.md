---
title: spring mvc校验
tags: [spring]
---

### 1. 校验注解结合国际化

1）spring mvc配置文件

```
<mvc:annotation-driven validator="validator"/>

<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
    <!-- 如果不加默认到 使用classpath下的 ValidationMessages.properties -->
    <property name="validationMessageSource" ref="messageSource" />
</bean>

<!-- 国际化的消息资源文件（本系统中主要用于显示/错误消息定制） -->
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basename" value="classpath:i18n/validmessage"></property>
    <property name="useCodeAsDefaultMessage" value="false" />
    <property name="defaultEncoding" value="UTF-8" />
    <property name="cacheSeconds" value="60" />
</bean>
```

2）实体类使用校验注解，在实体类的属性上添加注解

```
@NotNull(message="{user.username.notnull}")
@Pattern(regexp="^[a-zA-Z0-9]*$",message="{user.username}")
@Length(min=5,max=32,message="{user.username.len}")
private String username;
```

注：message如果要使用国际化资源文件中的key，必须使用大括号括起来。

注2：对于有占位符的国际化信息，占位符不能使用{0}，而必须使用{min}变量名称

3）在resources/i18n目录下添加资源文件

validmessage_zh_CN.properties(中文国际化资源文件)

```
user.username.notnull=用户名不空
user.username=用户名英文字母和数字的组合
user.username.len=用户名长度为{min}-{max}之间
```

注：validmessag.properties是默认的国际化资源文件，在找不到具体的国际化文件时，使用该文件。
