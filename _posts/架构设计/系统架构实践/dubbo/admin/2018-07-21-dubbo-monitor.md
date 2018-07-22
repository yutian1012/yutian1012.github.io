---
title: 使用dubbo搭建微服务架构-监控中心
tags: [architecture]
---

参考：https://www.jianshu.com/p/3d619740883c

参考2： http://dubbo.apache.org/#!/docs/admin/install/admin-console.md

dubbo官方提供了dubbo-admin子项目，主要用于路由规则、动态配置、服务降级、访问控制、权重调整、负载均衡等管理。

1）从官网下载（通过源码方式构建）

dubbo源码地址：https://github.com/apache/incubator-dubbo-ops

```
# 定位到待下载源码的目录下
git clone https://github.com/apache/incubator-dubbo-ops.git

# 修改配置文件
# 修改incubator-dubbo-ops\dubbo-admin\src\main\resources\application.properties
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
# 服务端口号
server.port=7001

# 编译打包
cd incubator-dubbo-ops && mvn package -Dmaven.test.skip=true
```

2）运行服务

dubbo-admin是使用spring-boot开发的项目，因此直接可以以java -jar的方式运行。

```
cd incubator-dubbo-ops\dubbo-admin\target
# 运行服务
java -jar dubbo-admin-0.0.1-SNAPSHOT.jar
```

3）访问

在application.properties文件中定义的服务端口为7001

```
http://localhost:7001
# 使用root用户登录，密码为root（application.properties文件中已配置）
```