---
title: maven父项目管理聚合多个子项目
tags: [maven]
---

参考：https://blog.csdn.net/qq_232911373/article/details/51381060

利用maven父项目来聚合多个子项目进行管理和组织。

1）eclipse中创建父项目

在创建父项目式选择package为pom，表明该maven项目是一个被继承的项目，即被子项目继承。

eclipse右键--new--projects--maven project

![](/images/tools/maven/project/mvn-parent-pom.png)

项目的GAV（group、artifact、version）

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.sjq.test</groupId>
  <artifactId>springdemo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
</project>
```

2）创建子项目

在父项目springdemo项目上右击--new--projects--maven modules

![](/images/tools/maven/project/mvn-sub-project.png)

项目的GAV

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.sjq.test</groupId>
    <artifactId>springdemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>jpademo</artifactId>
</project>
```

注：parent标签指定了父项目的GAV，表示继承该父项目。artifactId指定了子项目的名称，而groupId和version则省略了，这两个属性会自动继承父项目的两个标签值。

3）子项目的组织路径

创建子项目后，在eclipse中，子项目会在与父项目平级处出现相关的项目，同时在父项目的文件夹下也存在一份子项目。

![](/images/tools/maven/project/parent-sub-project.png)

注：修改时使用与父项目同级的子项目进行修改即可。

4）修改项目的编译版本

创建maven项目后，默认使用的jdk版本为1.5，可在父项目中修改编译的jdk版本信息，这样所有的子项目中都会同时更改。

在父项目的build标签中添加相应的编译插件，插件可以不用指定，因为maven默认对插件都会使用最新的版本进行构建。

```
<build>  
 <plugins>  
     <!--JDK版本 -->  
     <plugin>  
         <groupId>org.apache.maven.plugins</groupId>  
         <artifactId>maven-compiler-plugin</artifactId>  
         <configuration>  
             <source>1.8</source>  
             <target>1.8</target>  
             <encoding>UTF-8</encoding>  
             <showWarnings>true</showWarnings>  
         </configuration>  
     </plugin>  
 </plugins>  
</build>  
```

![](/images/tools/maven/project/jdk-compile-version.png)