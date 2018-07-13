---
title: maven 打包直接运行的spring boot项目
tags: [maven]
---

spring boot开发的项目可以以jar包的形式发布，并且能够直接通过java -jar的形式运行，内部可以嵌入tomcat作为发布容器，而无需使用打包成war包，然后部署到tomcat上。

1）添加依赖

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        ...
    </plugins>
</build>
```

注：这种方式非常适合微服务的发布项目