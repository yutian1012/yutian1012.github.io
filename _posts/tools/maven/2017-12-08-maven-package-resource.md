---
title: maven 打包资源文件
tags: [maven,tools]
---

参考：http://bglmmz.iteye.com/blog/2063856

最简单的方式是在pom.xml文件中添加打包的资源

```
<build>
    <resources>  
        <resource>  
            <directory>src/main/resources</directory>  
            <includes>  
                <include>**/*.properties</include>  
                <include>**/*.xml</include>  
                <include>**/*.tld</include>  
            </includes>  
            <filtering>false</filtering>  
        </resource>  
        <resource>  
            <directory>src/main/java</directory>  
            <includes>  
                <include>**/*.properties</include>  
                <include>**/*.xml</include>  
                <include>**/*.tld</include>  
            </includes>  
            <filtering>false</filtering>  
        </resource>  
    </resources>  
</build>
```