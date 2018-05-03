---
title: maven常用插件设置
tags: [maven,tools]
---

参考：http://blog.csdn.net/phantomes/article/details/51202005

maven插件配置时，一般需要知道版本号，默认会到中央仓库中下载最新的插件使用。

### maven插件

1）maven插件完成构件任务

Maven本质上是一个插件框架，它的核心并不执行任何具体的构建任务，所有这些任务都交给插件来完成，例如编译源代码是由maven-compiler-plugin完成的。

2）任务与目标

maven每个任务对应了一个插件目标（goal），每个插件会有一个或者多个目标，例如maven-compiler-plugin的compile目标用来编译位于src/main/java/目录下的主源码，testCompile目标用来编译位于src/test/java/目录下的测试源码。

3）插件的使用

maven可以通过两种方式调用Maven插件目标。

方式一：将插件目标与生命周期阶段（lifecycle phase）绑定，这样用户在命令行只是输入生命周期阶段而已，例如Maven默认将maven-compiler-plugin的compile目标与compile生命周期阶段绑定，因此命令mvn compile实际上是先定位到compile这一生命周期阶段，然后再根据绑定关系调用maven-compiler-plugin的compile目标。

方式二：直接在命令行指定要执行的插件目标，例如mvn archetype:generate 就表示调用maven-archetype-plugin的generate目标，这种带冒号的调用方式与生命周期无关。

4）插件生态圈

参考：http://maven.apache.org/plugins/index.html

Maven官方有多个插件列表，这些列表下列出来常用的一些插件。如列表的GroupId为org.apache.maven.plugins，这里的插件最为成熟，具体地址为：http://maven.apache.org/plugins/index.html。

另外列表的GroupId为org.codehaus.mojo，这里的插件没有那么核心，但也有不少十分有用，其地址为：http://www.mojohaus.org/plugins.html。

注：设置包括google提供的一些插件信息。

### 插件的获取

插件也是基于坐标保存在我们的Maven仓库当中的。在用到插件的时候会先从本地仓库查找插件，如果本地仓库没有则从远程仓库查找插件并下载到本地仓库。

1）插件的远程仓库

Maven会区别对待普通依赖的远程仓库与插件的远程仓库，不会使用配置好的普通依赖远程仓库，需要单独进行配置，使用pluginRepositories进行远程插件的配置。

```
<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>nexus</name>
        <url>http://192.168.0.70:8081/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
```

2）默认的远程插件仓库

Maven的父POM中也是有内置一个插件仓库的，${M2_HOME}/lib/maven-model-builder-3.0.4.jar，打开该文件，能找到超级父POM：\org\apache\maven\model\pom-4.0.0.xml，它是所有Maven POM的父POM，所有Maven项目都继承该配置。

```
<pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
</pluginRepositories>
```

注：默认插件仓库的地址就是中央仓库咯，它关闭了对snapshots的支持，防止引入snapshots版本的插件而导致不稳定的构件。

### 插件的定位与运行

问题，在执行mvn compiler:compiler命令时，是如何定位插件的（使用插件的版本号）

以编译插件maven-compiler-plugin为例，查看插件的运行过程。

1）项目中引入插件

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.7</source> <!-- 源代码使用的开发版本 -->
        <target>1.7</target> <!-- 需要生成的目标class文件的编译版本 -->
    </configuration>
</plugin>
```

2）执行命令

执行编译命令后会调用maven-compiler-plugin插件并执行compiler目标

```
mvn compiler:compiler
# 命令的完整写法应该是，groupId:artifactId:version:goal的形式调用插件
mvn org.apache.maven.plugins:maven-compiler-plugin:3.1:compiler
```

注：maven-compiler-plugin插件默认执行的目标为compiler。

3）查看Maven插件远程仓库的元数据

org/apache/maven/plugins/maven-metadata.xml，Maven默认的远程仓库是http://repo.maven.apache.org/maven2/。

a.获取插件的元数据定义信息

访问：http://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-metadata.xml

![](/images/tools/maven/maven-plugin-meta.png)

注：在元数据中会同prefix前缀指定对应的artifactId。即命令中的mvn compilder:compiler的第一个compiler就能够找到插件的artifactId了。

b.获取具体的compiler插件的元数据

访问：http://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-compiler-plugin/maven-metadata.xml

![](/images/tools/maven/maven-plugin-compiler-meta.png)

注：可以看到这里的最大版本是3.7.0

c.确定执行的插件版本

执行的是mvn compiler:compiler命令，由于maven-compiler-plugin的最新版本已经到了3.7.0，则默认会使用此版本。

d.总结

命令mvn compiler:compiler，第一个compiler是用于获取插件的artifactId，进而找到该插件发布的最大版本号。第二个compiler是插件的目标。