---
title: 微信通讯录同步
tags: [bpmx]
---

1）开启同步接口

通讯录同步相关接口，可以对部门和成员进行查询、添加、修改、删除操作。

登录企业微信--管理工具--开启API同步接口，并设置通讯录的编辑权限，完成后通过该Secret对通讯录的操作进行处理。

![](/images/weixin/qyweixin/department/syncdepartment.png)

2）系统中的同步功能

微信部门组织更新实现类WxOrgService，微信用户同步实现类WxUserService。
