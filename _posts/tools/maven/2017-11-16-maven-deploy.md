---
title: maven deploy项目到私库
tags: [maven,tools]
---

方式一：使用nexus软件上传第三方jar包（服务器端）

方式二：在私库服务器地址运行mvn的安装命令（服务器端）

```
mvn install:install-file -Dfile=D:\mvn\spring-context-support-3.1.0.RELEASE.jar -DgroupId=org.springframework -DartifactId=spring-context-support -Dversion=3.1.0.RELEASE -Dpackaging=jar
```

注： -Dfile=jar包的位置，-DgroupId=maven项目的groupId，-DartifactIdmaven项目的artifactId，Dversionmaven项目的version，-Dpackagingmaven项目的jar类型。

注2：install命令是安装jar包到本地仓库

方式三：客户端连接私库服务，使用install命令安装

参考：http://blog.csdn.net/Mr_rain/article/details/54022415

1）项目中引入私库，私库即可在项目的pom.xml中引入，也可在settings全局中引入

在项目的pom.xml中引入

```
<repositories>
    <repository>
        <id>nexus</id>
        <name>nexus</name>
        <url>http://172.17.30.102:8081/nexus/content/groups/public</url>
    </repository>
</repositories>
```

在settings.xml中引入私库，通过profile标签配置

```
<profiles>  
    
    <profile>  
      <id>nexus</id>  
      <repositories>  
        <repository>  
            <id>nexus</id>  
            <name>remote hotent maven release</name>  
            <url>http://172.17.30.102:8081/nexus/content/groups/public</url>  
        </repository>
      </repositories>  
    </profile>

    <!-- profile中配置了多个仓库，包括了发布版和快照版构件 -->
    <profile>
        <id>jboss</id>
        <repositories>
            <repository>
              <id>repository.jboss.org</id>
              <name>JBoss Repository</name>
              <layout>default</layout>
              <url>http://repository.jboss.org/maven2/</url>
              <snapshots>
                <enabled>false</enabled>
              </snapshots>
            </repository>
        
            <repository>
              <id>snapshots.jboss.org</id>
              <name>JBoss Snapshots Repository</name>
              <layout>default</layout>
              <url>http://snapshots.jboss.org/maven2/</url>
              <snapshots>
                 <enabled>true</enabled>
              </snapshots>
              <releases>
                 <enabled>false</enabled>
              </releases>
            </repository>
        </repositories>
    </profile>
</profiles>

...
<activeProfiles>
     <activeProfile>nexus</activeProfile>
     <activeProfile>jboss</activeProfile> 
</activeProfiles>
```

2）设置部署路径

在项目的pom.xml中引入

```
<!-- 指定部署的服务 -->
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>http://172.17.30.102:8081/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>http://172.17.30.102:8081/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

3）配置登录认证授权信息（发布到私库必须提供身份认证）

在settings.xml中添加认证信息

```
<servers>
    <server>
      <!-- id要与Nexus中的id对应上，同时与pom中指定的部署服务器id对应 -->
      <id>releases</id> 
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <!-- id要与Nexus中的id对应上，同时与pom中指定的部署服务器id对应 -->
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <!-- id要与Nexus中的id对应上，同时与pom中指定的部署服务器id对应 -->
      <id>3rd</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
</servers>
```

注：登录一个远程仓库需要用户名和密码进行身份验证，所以，需要远程仓库认证。配置认证信息和配置仓库信息不同，仓库信息可以直接配置POM文件中，但是认证信息必须配置在settings.xml文件中，目的就是安全性。

注2：这里的用户名和认证信息就直接使用了nexus的admin用户，没有创建新的用户。

4）运行命令部署

```
mvn deploy
```

注：将目录定位到项目文件夹下，该文件夹包含pom.xml

5）部署jar文件到第三方仓库

```
mvn deploy:deploy-file -DgroupId=com.xy.oracle -DartifactId=ojdbc14 -Dversion=10.2.0.4.0 -Dpackaging=jar -Dfile=E:\ojdbc14.jar -Durl=http://localhost:9090/nexus-2.2-01/content/repositories/thirdparty/ -DrepositoryId=thirdparty
```

注：这里指定了部署的远程仓库位置