---
title: maven设置项目的编译环境
tags: [maven,tools]
---

### 设置编译器版本

在pom.xml文件中添加maven-compiler-plugin

```
<build>
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