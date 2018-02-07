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

2）认证

向远程部署的构件往往需要认证，认证时创建server元素，其id要与仓库的id匹配。

3）部署到远程命令

```
mvn clean deploy
```

注：如果Maven项目当前的版本是快照版本，则部署到快照版本仓库地址，否则部署到发布版本仓库地址。