---
title: 持续集成jenkins配置邮件服务器
tags: [jenkins]
---

Jenkins使用JavaMail发送e-mail,JavaMail允许使用容器系统参数进行附加设置。

通过菜单：系统管理--系统设置，定位到最下方的邮件通知，配置SMTP服务器，并在高级设置中添加用户名和密码的认证，SMTP端口号，SSL协议等。

![](/images/tools/jenkins/jenkins-smtp.png)