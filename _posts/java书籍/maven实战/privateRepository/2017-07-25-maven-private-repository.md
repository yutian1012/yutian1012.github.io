---
title: maven私服搭建
tags: [maven]
---

参考：http://www.iteye.com/topic/1126678

第一步，在私服的setting.xml文件上配置多个Repository远程仓库，即私服代理的远程仓库（类型为Proxy）。

第二步，配置3个hosted repository，分别是3rd party、Snapshots、Releases，分别用来保存第三方jar（典型的比如ojdbc6.jar），项目组内部的快照、项目组内部的发布版。

第三步，创建一个group repository，group其实是一个虚拟的仓库，通过对实体仓库（proxy、hosted）进行聚合，对外暴露一个统一的地址。聚合上面配置的仓库。

第四步：配置用户认证信息

第五步：在用户端配置镜像，以及认证信息（~/.m2/settings.xml文件中配置）。

```
<mirrors>  
    <mirror>  
        <id>nexus</id>  
        <name>internal nexus repository</name>  
        <url>http://10.78.68.122:9090/nexus-2.1.1/content/groups/public/</url>  
        <mirrorOf>*</mirrorOf>  
    </mirror>  
</mirrors> 

<servers>  
    <server>  
        <id>nexus-snapshots</id>  
        <username>deployment</username>  
        <password>deployment</password>  
    </server>  
</servers> 
```

注：镜像地址为group repository对外公布的地址

第六步：配置maven项目的远程部署仓库

```
<!-- 配置部署的远程仓库 -->  
<distributionManagement>  
    <snapshotRepository>  
        <id>nexus-snapshots</id>  
        <name>nexus distribution snapshot repository</name>  
        <url>http://10.78.68.122:9090/nexus-2.1.1/content/repositories/snapshots/</url>  
    </snapshotRepository>  
</distributionManagement>
```

注：这里的远程部署仓库是我们在私服上创建的host仓库，用来保存maven项目的部署构件。