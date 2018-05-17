---
title: spring boot启动插件配置参数
tags: [spring]
---

参考：https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/maven-plugin/run-mojo.html

1）配置jvm参数

a.命令行中配置

```
mvn spring-boot:run -Drun.jvmArguments="-Xms512m -Xmx1024m"
```

b.pom.xml文件中配置

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <jvmArguments>-Xms512m -Xmx1024m</jvmArguments>
            </configuration>
        </plugin>
    </plugins>
</build>
```