---
title: maven部署至远程仓库
tags: [maven]
---

参考：http://www.cnblogs.com/AlanLee/p/6198413.html

私服的一大作用是部署第三方构件，包含组织内部生成的构件以及一些无法从外部仓库直接获取的构件。

1）配置部署到远程仓库

编辑pom.xml文件，配置distributionManagement元素

```
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>public</name>
        <url>http://59.50.95.66:8081/nexus/content/repositories/releases</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Snapshots</name>
        <url>http://59.50.95.66:8081/nexus/content/repositories/snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

注：repository元素表示发布版本构件的仓库，snapshotRepository元素表示快照版本的仓库。