---
title: maven 仓库和镜像
tags: [maven,tools]
---

参考：http://blog.csdn.net/u011768325/article/details/49735397

1）镜像的概念

如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像，某些情况下使用镜像可以提高项目构建效率。

镜像的意思是，当你访问mirrorOf的仓库时，就会转到对应的镜像url中请求对应的资源。

2）settings.xml 中配置镜像

```
<mirrors>
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
</mirrors>
```

注：当访问repositoryId仓库时都会被拦截，并交由mirrorId配置的url来获取。

3）镜像的实际使用

一般镜像都是和私服结合使用。由于私服可以代理任何外部的公共仓库（包括中央仓库），因此，对于组织内部的Maven用户来说，使用一个私服地址就等于使用了所有需要的外部仓库，这个可以将配置集中到私服，从而简化Maven本身的配置。在这种情况下，任何需要的构件都可以从私服中获得，私服就是所有仓库的镜像。

```
<mirrors>
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>*</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
</mirrors>
```

4）镜像实际应用2

节省网络传输，针对中央仓库外网访问慢的特点使用镜像优化。http://maven.net.cn/content/groups/public/ 是中央仓库http://repo1.maven.org/maven2/ 在中国的镜像，由于地理位置的因素，该镜像往往能够提供比中央仓库更快的务。因此，可以配置Maven使用该镜像来替代中央仓库。编辑settings.xml

```
<mirrors>
    <mirror>
      <id>maven.net.cn</id>
      <name>one of the central mirrors in china</name>
      <url>http://maven.net.cn/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

注：这里的mirrorOf值为central，即作为central的代理镜像。

注2：通过这个例子，可以做到手动的更改检索仓库位置的作用。同样，也可通过在settings中再次定义central仓库，覆盖默认的仓库设置来更改检索仓库位置。