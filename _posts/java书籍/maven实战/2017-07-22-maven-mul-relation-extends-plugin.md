---
title: maven多模块关联--继承的插件管理
tags: [maven]
---

maven提供了dependencyManagement元素帮助管理依赖，类似的Maven也提供了pluginManagement元素帮助管理插件。在该元素中配置的依赖不会造成实际的插件调用行为。当POM中配置了真正的plugin元素，并且其groupId和artifactId与pluginManagement中配置的插件匹配时，pluginManagement的配置才会影响实际的插件行为。

### 父模块中定义插件

1）父模块下配置pluginManagement元素

当项目中的多个模块有同样的插件配置时，应当将配置移动到父POM的pluginManagement元素中。即使各个模块对于同一插件的具体配置不尽相同，也应当使用父POM的pluginManagement元素统一声明插件的版本。

account-email和account-persist子模块都使用了maven-compiler-plugin插件，并且制定了编译使用的jdk版本信息。应将其移动到父模块的pluginManagement元素下

```
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.5</source>
                    <target>1.5</target>
                </configuration>
            </plugin>
           ...
        </plugins>
    </pluginManagement>
</build>
```

注：甚至可以要求将所有用到的插件的版本在父POM的pluginManagement元素中声明，子模块使用时不配置版本信息，这么做可以统一项目的插件版本，避免潜在的插件不一致或不稳定问题，也更易于维护。

2）子模块引入插件

在account-emial子模块中引入插件

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
        </plugin>
        ...
    </plugins>
<build>
```

注：子模块声明使用maven-compiler-plugin插件，同时又继承了父模块的pluginManagement配置，两者基于groupId和artifactId匹配合并之后就相当于下面完整插件信息的配置。

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.5</source>
        <target>1.5</target>
    </configuration>
</plugin>
```

### 插件的默认配置

1）插件的默认继承机制

在父模块account-parent中移除maven-compiler-plugin的版本信息，并在子模块account-email中移除maven-compiler-plugin的插件配置。

account-parent项目的pluginManagement元素（移除了version）：

```
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
           ...
        </plugins>
    </pluginManagement>
</build>
```

account-email模块的插件配置

```
<build></build>
```

注：该子模块不再配置插件信息

account-email依然能够享受到maven-compiler-plugin插件的服务，maven-compiler-plugin在父模块中开启了java5编译的支持。首先，内置的插件绑定关系将maven-compiler-plugin插件绑定到了account-email的生命周期上；其次，超级POM为maven-compiler-plugin插件声明了版本信息；最后account-parent中的pluginManagement对maven-compiler-plugin的行为进行了配置。

2）超级POM的位置

安装maven后，在安装目录下%M2_HOME%/lib/maven-model-builder-x.x.x.jar中存在超级POM，访问路径org/apache/maven/model/pom-4.0.0.xml文件。