---
title: github开源项目云收藏（邮箱配置）
tags: [project]
---

1）引入依赖jar文件

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

2）修改配置文件

```
spring.mail.host=smtp.126.com
spring.mail.username=yutian1012@126.com
spring.mail.password=xxxx
spring.mail.default-encoding=UTF-8
mail.fromMail.addr=yutian1012@126.com
```

3）发送邮件相关类

a.JavaMailSender类

最早期使用JavaMail相关api来写发送邮件的相关代码，JavaMailSender是spring对JavaMail相关api的封装，使用该类更加简化了邮件发送的过程。

b.SimpleMailMessage类

该类用于封装文本邮件的发送内容，指定发件人，收件人以及相关的发送邮件内容。

c.MimeMessage类

该类支持html样式的邮件以及附件的邮件。

```
//使用JavaMailSender类创建出MimeMessage对象
MimeMessage message = mailSender.createMimeMessage();
//借助MimeMessageHelper封装信息到MimeMessage中
MimeMessageHelper helper = new MimeMessageHelper(message, true);
```