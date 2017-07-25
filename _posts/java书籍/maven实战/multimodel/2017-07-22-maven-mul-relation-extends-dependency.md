---
title: maven多模块关联--继承的依赖管理
tags: [maven]
---

依赖是会被继承的。account-parent作为父模块，子模块account-email和account-persist模块同时依赖了很多相同的jar包（spring-core,spring-beans等），可以将这些依赖配置放到父模块account-parent中，两个子模块就能够移除这些配依赖，简化配置。

1）存在的问题

如果将两个子模块都需要的jar包的依赖移动到父模块中，然后再从父模块继承回来，对这两个模块是没有影响到。假设将来需要加入一个account-util模块，该模块只是提供了一些简单的帮助工具，与springframework完全无关，难道也要继承依赖spring-core等jar包吗？

2）父模块下配置dependencyManagement元素

该元素即能让子模块继承到父模块的依赖配置，又能保证子模块依赖使用的灵活性。dependencymanagement元素下的依赖声明不会引入实际的依赖，不过它能够约束dependencies下的依赖使用。

在account-parent模块下添加dependencyManagement

```
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
        ...
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

不仅消除了一些重复，也能够使得各依赖的版本处于更加明显的位置。这段配置虽然不会引入依赖，但却会被继承。在子模块的effectivepom（eclipse插件）中能够查看详细的配置信息（包括从父模块中继承的配置信息）。

3）子模块引入依赖

account-parent模块中已经定义了完整的依赖声明（dependencyManagement元素），子模块只需要配置简单的groupId和artifactId就能获得对应的依赖信息，从而引入正确的依赖。

使用这种依赖管理机制似乎不能减少太多的POM配置，但dependencyManagement声明依赖能够统一项目范围中依赖的版本。当依赖版本在父POM中声明之后，子模块在使用依赖的时候就无须声明版本，也就不会发生多个子模块使用依赖版本不一致的情况。这可以帮助降低依赖冲突的几率。

account-email模块引入依赖

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>        
    </dependency>
</dependencies>
```

注：引入依赖时不需要添加版本信息，依赖的版本信息已经在父模块中声明了。

4）子模块间的依赖

account-service模块用来封装account-email、account-persist等模块的实现细节，因此它肯定需要依赖这三个模块。

```
<dependency>
    <groupId>${project.groupId}</groupId>
    <artifactId>account-email</artifactId>
    <version>${project.version}</version>
</dependency>
```