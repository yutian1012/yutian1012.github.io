---
title: maven 打包spring boot项目成普通jar
tags: [maven]
---

前提：项目是以maven moudel的形式创建了多个moudel，其中有一个moduel是作为api，可提供给外部客户端调用的jar。现希望，将其单独打包成jar，并发生给客户端直接依赖。

1）maven父项目的配置

这里只看打包配置

```
<build>
    <finalName>xxxx</finalName>
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

可以看到，这里直接使用的是spring-boot-maven-plugin插件，在打包时会使用该插件提供的打包命令。

2）对单独module进行打包错误信息

```
xxx :repackage failed: Unable to find main class -> [Help 1]
```

注：这里不需要打包成可运行的jar

3）解决方法

不要在父项目的中配置spring-boot-maven-plugin插件，如果想要重用该插件配置，可以设置pluginManagement。

```
<build>
    <finalName>xxxx</finalName>
    <pluginManagement>
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
    </pluginManagement>
</build>
```