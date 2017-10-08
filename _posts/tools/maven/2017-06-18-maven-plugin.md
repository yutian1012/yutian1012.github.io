---
title: maven常用插件设置
tags: [maven,tools]
---

参考：http://blog.csdn.net/phantomes/article/details/51202005

### 设置编译器版本

1）编译版本

在pom.xml文件中添加maven-compiler-plugin插件，并设置编译版本

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

注：使用插件maven-compiler-plugin可设置maven项目的编译器版本。

如果在IDE环境下，可以右击项目--maven--update project..

2）另外，如果maven编译时出现乱码，可指定编码

maven-compiler-plugin插件的configuration中可以指定编译时代码的编码。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.7.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>utf-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

注：通过encoding标签来指定编码。

### 生成源码包和文档包

maven源码包是通过maven-source-plugin插件实现的，文档包是通过maven-javadoc-plugin实现的。

1）打包时同时生成源码，在项目的pom.xml中配置

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>3.0.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

注：在执行mvn package命令时，会自动生成编译后的jar包和源代码的jar包。

2）直接使用命令方式生成源码包

```
mvn source:jar
```

使用上面的方式也可以生成源码包，实际运行也是借助maven-source-plugin来生成的。

3）在pom.xml中配置文档生成的插件

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.10.4</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

或者通过命令的方式运行

```
mvn javadoc:javadoc
```

### maven测试插件

maven的单元测试目录位于${project.basedir}/src/test/java，使用插件maven-surefire-plugin来完成单元测试任务。该插件的配置同样可以在pom.xml或单独使用命令来运行。

注：maven在执行package时，默认会执行单元测试，并生成测试报告。

单元测试的过程是很缓慢的，如果不希望执行单元测试，可直接配置跳过单元测试的执行

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.20.1</version>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```

或者使用命令行的方式跳过单元测试

```
mvn package -Dmaven.test.skip=ture
```

### maven打包插件

使用maven-assembly-plugin可实现项目的打包，需要配置一个assembly.xml，来指定打包的目录信息和文件存放信息。

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <finalName>mylib</finalName>
        <appendAssemblyId>false</appendAssemblyId>
        <encoding>utf-8</encoding>
        <descriptors>
          <descriptor>src/main/assembly/assembly.xml</descriptor>
        </descriptors>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

注：使用该插件时，需要设置assembly，即装配方式。

assembly.xml文件用来指定打包时的目录组织和文件存放信息。

参考：http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html

```
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
    <id>distribution</id>
    <!-- 打包成zip文件 -->
    <formats>
        <format>zip</format>
    </formats>

    <fileSets>
        <!-- 将项目下的src/main/resources目录下的文件打包到压缩文件的根目录下-->
        <fileSet>
            <directory>${project.basedir}\src\main\resources</directory>
            <outputDirectory>\</outputDirectory>
        </fileSet>
        <!-- 将项目下的src/bin目录下的class文件打包到压缩文件的/bin目录下 -->
        <fileSet>
            <directory>${project.basedir}\src\bin</directory>
            <outputDirectory>\bin</outputDirectory>
        </fileSet>
    </fileSets>
    <!-- 相关依赖信息打包的压缩包的/lib目录下 -->
    <dependencySets>
        <dependencySet>
            <useProjectArtifact>true</useProjectArtifact>
            <outputDirectory>lib</outputDirectory>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
</assembly>
```