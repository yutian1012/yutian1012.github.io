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

2）在springclouddemo父模块中添加插件配置

```
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
```

注：executable指打包后生成可执行jar包，当使用spring-boot-dependencies方式时，需要配置executions标签。

运行打包命令（这里要使用spring-boot提供的插件运行打包命令）

```
# 将依赖打包到同一个jar文件中
mvn clean package -Dmaven.test.skip=true
# 本地先测试运行
jar -jar xxxx.jar
```

3）问题：SpringBoot打包成jar后运行提示没有主清单属性

参考：https://www.cnblogs.com/niceboat/p/6230448.html

解决方法：在插件中配置executions标签信息

```
<executions>
    <execution>
        <goals>
            <goal>repackage</goal>
        </goals>
    </execution>
</executions>
```

原因：项目中使用的是spring-boot-dependencies这个POM，而不是spring-boot-starter-parent这个POM。导致spring-boot-maven-plugin的配置项丢失，使得打包后的jar中的MANIFEST.MF文件缺少Main-Class。

4）修改生成的jar包名称

microservicecloud-eureka-7001模块打包后生成的名称为microservicecloud.jar，即继承了父类定义的finalName，这里需要修改自己模块名称，查看下面的配置信息。

```
<build>
    <finalName>microservicecloudEureka7001</finalName>
</build>
```

5）上传jar文件到linux系统中

使用fileZilla即可上传文件（使用SFTP连接SHH端口）。

6）在linux系统上启动

定位jar文件的位置，运行如下命令

```
# 在当前窗口中启动，同时信息也会输出到该窗口中
java -jar xxxx.jar
# nohup方式运行，并忽略所有输出
nohup java -jar xxx.jar >/dev/null &

# 查看是否启动
ps -ef | grep java
```

7）页面访问

直接访问阿里云服务器，发现无法访问，原因是入网端口没有打开。需要到aliyun服务后台开放相应的端口。这里使用的是7001端口，因此开启7001的入网端口。

云服务器ECS--实例--管理，进入实例管理页面，本实例的安全组--配置规则

![](/images/weixin/aliyun/port/manage-instance-security-list.png)