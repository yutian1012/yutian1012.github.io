---
title: 将eureka部署到linux系统上
tags: [spring]
---

由于本地环境配置太低，服务运行多个实例，现将服务部署到linux服务上，进行测试。

1）打包jar

针对microservicecloud-eureka-7001模块，进行打包上传

```
# 定位到microservicecloud-eureka-7001模块目录下，运行
mvn clean package
```

注：包大小只有4KB，很显然没有将依赖打入包中，这样在linux下是不能直接运行的

在springclouddemo父模块中添加插件配置

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.4</version>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

运行打包命令，生成microservicecloud-jar-with-dependencies.jar文件

```
# 将依赖打包到同一个jar文件中
mvn assembly:assembly -DskipTests
```