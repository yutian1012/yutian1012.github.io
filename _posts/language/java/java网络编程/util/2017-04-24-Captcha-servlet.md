---
title: SimpleCaptcha验证码
tags: [web-util]
---

```
<servlet>
    <servlet-name>StickyCaptcha</servlet-name>
    <servlet-class>nl.captcha.servlet.SimpleCaptchaServlet</servlet-class>
    <init-param>
        <param-name>captcha-width</param-name>
        <param-value>250</param-value>
    </init-param>
    <init-param>
        <param-name>captcha-height</param-name>
        <param-value>75</param-value>
    </init-param>
</servlet>
```

参考：http://lj-zhu.iteye.com/blog/1028970

在pom.xml中添加依赖

```
<!-- captcha -java.lang.ClassNotFoundException: com.jhlabs.image.RippleFilter -->
<dependency>
    <groupId>com.jhlabs</groupId>
    <artifactId>imaging</artifactId>
    <version>01012005</version>
</dependency>

<!-- 指定仓库地址 -->
<repositories>
    <repository>  
        <id>atlassian</id>  
        <name>atlassian</name>  
        <url>https://maven.atlassian.com/3rdparty/</url>  
    </repository> 
</repositories>
```

