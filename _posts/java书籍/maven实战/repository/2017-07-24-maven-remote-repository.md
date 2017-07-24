---
title: maven远程仓库配置
tags: [maven]
---

有时候默认的中央仓库无法满足项目的需求，可能项目需要的构件存在于另外一个远程仓库中，如JBoss maven仓库，可以在POM中配置该仓库。

1）配置远程仓库

```
<repositories>
    <repository>
        <id>jboss</id>
        <name>JBoss Repository</name>
        <url>http://repository.jboss.com/maven2/</url>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>daily</updatePolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
            <checksumPolicy>warn</checksumPolicy>
        </snapshots>
        <layout>default</layout>
    </repository>
</repositories>
```

注：任何一个仓库声明的id必须是唯一的，尤其需要注意到是，Maven自带的中央仓库使用的id为central，不要覆盖了中央仓库的配置。

在releases标签下enabled元素为true，表示开启JBoss仓库的发布版本下载支持，而snapshots的enabled值为false，标识关闭JBoss仓库的快照版本的下载支持。

2）远程仓库的认证

有时候处于安全方面的考虑，需要提供认证信息才能访问远程仓库。一般私服的访问都需要提供认证。

配置认证信息和配置仓库信息不同，仓库信息可以直接配置在POM文件中，但是认证信息必须配置在settings.xml文件中，这是因为POM往往是被提交到代码仓库中供所有成员访问的。而settings.xml一般放在本机，因此在settings.xml中配置认证信息更安全。

```
<servers>
    <server>
        <id>my-proj</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```

server元素配置仓库认证信息，id元素必须与POM中需要认证的repository元素的id完全一致。换句话说，正是这个id将认证信息与仓库配置关联在了一起。