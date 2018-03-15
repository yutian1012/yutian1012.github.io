---
title: saiku源码导入eclipse
tags: [BI]
---

参考：http://blog.csdn.net/gsying1474/article/details/51603535

项目的源码：https://github.com/lightingLYG/saiku3

1）克隆项目到本地

```
git clone https://github.com/lightingLYG/saiku3.git
```

2）导入MyEclipse

错误提示：Versions of Spring facet could not be detected。

注：直接忽略掉该错误，cancel即可。

3）修改项目中引用的仓库地址

项目中引用的是自己搭建的maven私库，这里我们只能自己处理，将下载的可运行的saiku程序，获取程序的jar安装到自己的本地maven仓库中，这样就不会到外网再去检索jar包了。

4）修改saiku-beans.properties文件

```
修改所有"../../"的地方替换为"../webapps/saiku3/"

修改pluginpath为"../webapps/saiku3/js/saiku/plugins/"
```

5）修改pom.xml文件

```
# 隐藏batik
<!-- <dependency>
    <groupId>batik</groupId>
    <artifactId>batik-transcoder</artifactId>
    <version>1.7</version>
</dependency> -->
<!-- <dependency>
    <groupId>batik</groupId>
    <artifactId>batik-ext</artifactId>
    <version>1.7</version>
</dependency> -->

# 修改版本号
<dependency>
    <groupId>org.saiku</groupId>
    <artifactId>saiku-query</artifactId>
    <version>0.4-SNAPSHOT</version>
</dependency>

# 修改hsql
<!-- <dependency>
    <groupId>mondrian-data-foodmart-hsql</groupId>
    <artifactId>mondrian-data-foodmart-hsql</artifactId>
    <version>0.1</version>
</dependency> -->
<dependency>
    <groupId>pentaho</groupId>
    <artifactId>mondrian-data-foodmart-hsqldb</artifactId>
    <version>0.2</version>
</dependency>
```

6）修改报错文件

org.saiku.olap.query2.util.Fat类

```
CalculatedMember cm =new CalculatedMember(h.getDimension(),h,qcm.getName(),
  null,
  null,
  Member.Type.FORMULA,
  qcm.getFormula(),
  qcm.getProperties(),true);
```

注：在末尾添加一个ture即可

7）部署并试运行

```
http://localhost:8080/saiku3/
```

使用用户名admin，密码admin访问系统。