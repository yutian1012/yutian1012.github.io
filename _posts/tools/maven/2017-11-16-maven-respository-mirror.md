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

### 仓库和镜像的关系

参考：https://www.cnblogs.com/xdouby/p/6502925.html

1）仓库的配置

在pom.xml里的repositories元素，里面可以包含多少repository（至少默认包含了中央仓库，仓库id为central，如果写id为central的mirror或者repository覆盖默认的中央仓库，该仓库总是在effective-pom里repositories元素的最后一个子元素），每个repository都有一个id(此id非常重要)

```
mvn help:effective-pom //验证central仓库出现的位置
```

注：从超级父pom里继承来的中央repository在effective-pom里总是为最后一个repository

2）仓库与镜像的查找关系

Maven获取真正起作用的repository集合流程：

首先会获取pom.xml里的repository集合，然后在settings.xml里找mirrors元素，如果repository的id和mirror的mirrorOf的值相同，则该mirror替代该repository，如果该repository找不到对应的mirror，则使用其本身，依此可以得到最终起作用的repository集合。

3）查找依赖jar包的过程

关于maven如何查找pom.xml里dependencies里配置的插件，暂且不考虑本地仓库的存在，maven会根据最终的repository集合里依次查找，如果查到了就从该仓库下载，并且停止对后续repository的查找（找到了就停）。所以可以看出用户在pom.xml里配置repository，repository的顺序还是挺重要的。

注：实际依赖的查找，应该是先查找本地的仓库，如果本地仓库查找不到，再通过repository里面配置的仓库进行查找。

