---
title: maven多模块关联--继承
tags: [maven]
---

项目场景（简单的账号注册服务）：用户注册后，通过提供的邮箱进行激活，并登录。

注：参考第4章的项目信息介绍

### 介绍

多模块间主要存在模块之间的继承关系和依赖关系（参考maven实战的第8章）。该背景案例最终会被划分为account-email，account-persist等五个模块。Maven的聚合特性能够把项目的各个模块聚合在一起构建，而Maven的继承特性则能帮助抽取各个模块相同的依赖和插件等配置，简化并规范POM，促进了各个模块的一致性。

### 模块间的继承

1）account-email模块的pom文件

account-email项目，是案例中的一个email模块，负责发送账户激活的电子邮件（代码参考第5章）。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juven.mvnbook.account</groupId>
    <artifactId>account-email</artifactId>
    <name>Account Email</name>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>2.5.6</version>
        </dependency>
       ...
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.5</source>
                    <target>1.5</target>
                </configuration>
            </plugin>
            ...
        </plugins>
    </build>

</project>
```

注：为了减少篇幅，这里省略了依赖和插件的部分配置代码。

2）account-persist模块的pom文件

该模块负责账号数据的持久化，以XML文件的形式保存账号数据，并指出账号的创建、读取、更新、删除等操作。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-persist</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <name>Account Persist</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>2.5.6</version>
        </dependency>
        ...
    </dependencies>

    <build>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.5</source>
                <target>1.5</target>
            </configuration>
        </plugin>
        ...
    </build>
</project>
```

注：该模块groupId和version与account-email模块完全一致（这也是其他子模块存在的重复性问题），而且artififactId也有相同的前缀。另外依赖的配置和插件的配置也有很大一部分是重复的。

一般来说，一个项目的子模块都应该使用相同的groupId，如果它们一起开发和发布，还应该使用同样的version，此外，它们的artifactId还应该使用一致的前缀，以方便同其他项目区分。

3）继承

在上述的两个模块account-email和accout-persist中的两个pom中有着很多相同的配置，如相同的groupId和version，有相同的spring-core，spring-beans，spring-context和junit依赖，还有相同的maven-compile-plugin与maven-resources-plugin配置。重复往往意味着更多的劳动和更多的潜在问题。在maven中也存在于java对象继承类似的机制，可以抽取出重复的配置。

4）创建account-parent的maven项目（父模块）

注意该模块的pom文件的packaging元素值为pom，即打包类型为pom。该父模块是为了消除配置的重复，因此它本身不包含除POM之外的项目文件（即只有pom.xml文件），不需要src/main/java之类的文件夹。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
    http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <packaging>pom</packaging>
    <name>Account Parent</name>

    <properties>
        <springframework.version>2.5.6</springframework.version>
        <junit.version>4.7</junit.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${springframework.version}</version>
            </dependency>
           ...
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <source>1.5</source>
                        <target>1.5</target>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                    <configuration>
                        <encoding>UTF-8</encoding>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>    
</project>
```

5）改造account-email模块，继承父模块

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.juvenxu.mvnbook.account</groupId>
        <artifactId>account-parent</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../account-parent/pom.xml</relativePath>
    </parent>
    
    <artifactId>account-email</artifactId>
    <name>Account Email</name>

    <properties>
        <javax.mail.version>1.4.1</javax.mail.version>
        <greenmail.version>1.3.1b</greenmail.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>        
        </dependency>
        ...
        <dependency>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
            <version>${javax.mail.version}</version>
        </dependency>       
        ...
    </dependencies>
    <build>
    </build>

</project>
```

parent元素中声明父模块，指明父模块的坐标

relativePath元素指明父模块pom的相对路径，因为account-email模块与account-parent模块是平行目录的关系，因此这里的relativePath的值为../account-parent/pom.xml，该属性的默认值为../pom.xml。找不到时会从本地仓库查找。

account-email模块没有声明groupId和version等信息，会直接从父类中继承，声明了就会覆盖掉从父类继承的值。

6）relativePath属性

一般情况下，开发团队的新成员从源码库签出一个包含父模块关系的Maven项目，由于只关心其中的某一个子模块，就直接到该模块目录下执行构建，这个时候父模块是没有被安装到本地仓库，因此如果子模块没有设置正确的relativePath，Maven将无法找到父POM，将导致构建失败。