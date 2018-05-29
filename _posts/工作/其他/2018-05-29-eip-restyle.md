---
title: 替换系统的文字和样式
tags: [work]
---

场景：产品出售给一家企业，需要将产品中的系统名称等属性替换成该企业相关信息。同时替换相应的样式。

1）修改系统属性

```
# 附件存放地址
file.upload     c:/uip/upload
# 系统名称
appName     xxx系统
# 备份地址
backup.address  c:/uip/backup/db
```

2）修改子系统名

修改子系统名称，访问地址：/platform/system/subSystem/list.ht

3）修改logo图片

```
# 登录系统显示的图片
/eip-saas/styles/login/images/logo.gif
# 进入系统展示的图片
/eip-saas/styles/default/images/resicon/my/logo.png
```

4）隐藏相关的连接

访问：系统配置--资源管理