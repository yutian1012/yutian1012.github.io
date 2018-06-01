---
title: 阿里云服务器-安全管理组配置开发端口
tags: [weixin]
---

在阿里云上部署了一个服务，对外暴露7001端口，服务启动后，无法访问。

原因：阿里云服务器没有开启7001端口的访问权限。

解决方法：在安全组中配置安全规则，配置入网规则中开启7001端口

1）菜单设置

云服务器ECS--实例--管理，进入实例管理页面

![](/images/weixin/aliyun/port/manage-instance.png)

左侧菜单：本实例的安全组--配置规则

![](/images/weixin/aliyun/port/manage-instance-security.png)

右上角快速创建规则按钮，规则方向选择入网（因为是由外部发来请求

```
授权策略      协议类型      端口范围        授权类型        授权对象
允许          自定义TCP    7001/7001      地址段访问     0.0.0.0/0

# 授权对象0.0.0./0表示不显示任何地址段的访问
# 端口范围7001/7001，需要严格按照格式，使用/分隔端口范围
```

![](/images/weixin/aliyun/port/manage-instance-port.png)

结果会在入方向列表中增加一条数据

![](/images/weixin/aliyun/port/manage-instance-security-list.png)