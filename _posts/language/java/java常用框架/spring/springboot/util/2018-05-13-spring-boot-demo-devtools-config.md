---
title: spring boot devtools的常用配置
tags: [spring]
---

1）排除不需要监控的资源变动

某些资源在更改时不一定需要触发重启。如直接编辑Thymeleaf模板。默认情况下，更改/META-INF/maven，/META-INF/resources，/resources，/static，/public或/templates中的资源不会触发重启，但会触发实时重新加载。

如果要自定义这些排除项，可以使用spring.devtools.restart.exclude属性。

如果你想保留上面的默认（情况下的）值并添加其他的排除项，你可以使用spring.devtools.restart.additional-exclude属性。

编辑application.properties文件，多个值使用逗号分隔

```
spring.devtools.restart.exclude=static/**,public/**

spring.devtools.restart.additional-exclude=...
```

2）监控额外的路径

当对不在类路径中的文件进行更改时，可能需要重启或重新加载应用程序。为此，请使用spring.devtools.restart.additional-paths属性来配置监视其他路径的更改。

```
spring.devtools.restart.additional-paths=...
```