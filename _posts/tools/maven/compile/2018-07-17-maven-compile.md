---
title: maven设置编译器版本
tags: [maven,tools]
---

使用maven-compiler-plugin插件设置编译版本

```
<build>
    <finalName>webmodel</finalName>
    <pluginManagement>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
            </configuration>
        </plugin>
    </plugins>
   </pluginManagement>
</build>
```