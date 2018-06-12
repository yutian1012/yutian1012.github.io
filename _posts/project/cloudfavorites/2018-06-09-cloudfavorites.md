---
title: github开源项目云收藏（本地下载与运行）
tags: [project]
---

感谢纯洁的微笑提供的开源项目，该项目可以用来高效的学习搭建spring boot项目。

参考：https://github.com/cloudfavorites/favorites-web

注：学习过程采用从无到有的搭建项目，并参考代码的实现，来达到高效的学习目的。

1）下载源码

```
# 下载最新版本
cd E:/work/gitproject
git clone https://github.com/cloudfavorites/favorites-web.git
# 切换到最新版本
cd favorites-web
git checkout favorites-1.2.2
git status
```

2）在eclipse中查看源码

导入项目到eclipse中，定位到app目录下，该目录下存在pom.xml文件

菜单：File--import--exists maven project，然后选择所要导入的maven项目。

3）修改数据库配置

查看配置文件application.properties

```
# 指定了激活的环境变量
spring.profiles.active=dev
```

查看文件application-dev.properties，该文件提供了数据源的配置

```
spring.datasource.url=jdbc:mysql://127.0.0.1/favorites?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

注：这里可以修改数据库的配置信息

创建数据库

```
CREATE DATABASE IF NOT EXISTS favorites default character set utf8 COLLATE utf8_general_ci;
```

4）启动项目并试运行

在eclipse中运行main方法的形式运行，运行Java类FavoritesApplication，该类是spring boot服务的入口类。

5）使用功能

注册用户并登录系统，按照提供的gif动画进行操作（查看网页收集工具页面）

第一步：将云收藏放置到工具栏，浏览器网页。

第二步：定位需要收藏的页面，点击工具栏中的云收藏，将该网页加入到云收藏进行管理。

第三步：创建收藏夹，可以将未读列表中的收藏信息转移到新创建的收藏夹中。

6）提供的功能点

个人信息；邮件提醒信息；系统配置信息；收藏夹；未读列表；回收站；评论；点赞；导入/导出收藏夹；定时器；

注：针对每一个功能点单独分析创建过程，来实现spring boot功能。