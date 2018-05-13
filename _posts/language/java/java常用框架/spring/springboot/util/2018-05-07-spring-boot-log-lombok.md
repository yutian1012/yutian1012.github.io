---
title: spring boot配置使用lombok
tags: [spring]
---

参考：https://blog.csdn.net/linfeng_csdn/article/details/77189223

使用lombok是需要安装的，如果不安装，IDE则无法解析lombok注解。

1）引入依赖

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

引入依赖后，项目使用mvn能够正常编译通过，但在ide环境下却不能正确被使用

2）ide环境下的使用

添加依赖后，在相应的仓库下就会存在该jar包。双击运行该jar文件，会弹窗一个对话框，该对话框能够正确的识别系统中的ide，并进行这些ide进行插件的安装。

如本地的mvn仓库下的lombok.jar文件，也可也通过网页的形式下载该jar文件。

网页下载地址：https://projectlombok.org/download

```
G:\repository\org\projectlombok\lombok\1.16.20\lombok-1.16.20.jar
```

![](/images/spring/springboot/spring-boot-lombok.png)

注：安装完成后重新启动IDE。

3）eclipse安装lombok

以eclipse为例，运行lombok-1.16.20.jar文件后，会在eclipse的安装目录下出现lombok.jar文件，同时eclipse.ini文件的末尾追加了一行文本。

```
-javaagent:G:\eclipse\lombok.jar
```

![](/images/spring/springboot/spring-boot-lombok-installed.png)

4）常用注解

a.@Data注解

该注解应用在类上，提供类所有属性的getter和setter方法，此外还提供了equals、canEqual、hashCode、toString方法。

b.@Setter注解

该注解应用在属性或类上，为属性提供setter方法

c.@Getter注解

该注解应用在属性或类上，为属性提供getter方法

d.@Log4j注解

该注解应用在类上，为类提供一个属性名为log的log4j日志对象

e.@NoArgsConstructor注解

该注解应用在类上，为类提供一个无参的构造方法

f.@AllArgsConstructor注解

该注解应用在类上，为类提供一个全参的构造方法