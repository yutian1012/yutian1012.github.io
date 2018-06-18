---
title: github开源项目云收藏（定义邮件发送模板）
tags: [project]
---

有些邮件内容基本是固定的，只需要对部分字段替换即可，这部分邮件可以定义为邮件模板。

1）使用thymeleaf定义邮件模板

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2）在resorces/templates下创建emailTemplate.html

```
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8"/>
        <title>Title</title>
    </head>
    <body>
        好,这是验证邮件,请点击下面的链接完成验证,<br/>
        <a href="#" th:href="@{ http://www.xxx.com/neo/{id}(id=${id}) }">激活账号</a>
    </body>
</html>
```

3）定义邮件发送接口

```
package com.favorites.favoriteswebdemo.service;

import java.util.Map;

public interface MailService {
    public void sendTemplateMail(String templateName,Map<String,Object> params,String to, String subject);
}
```

4）定义实现类

```
package com.favorites.favoriteswebdemo.service;

import java.io.File;
import java.util.Map;

import javax.annotation.Resource;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Component;
import org.thymeleaf.context.Context;
import org.thymeleaf.spring5.SpringTemplateEngine;

@Component
public class MailServiceImpl implements MailService{
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Resource
    private JavaMailSender mailSender;
    
    @Resource
    private SpringTemplateEngine springTemplateEngine;

    @Value("${mail.fromMail.addr}")
    private String from;

    @Override
    public void sendTemplateMail(String templateName,Map<String,Object> params,String to, String subject) {
        Context context = new Context();
        if(null!=params) {
            context.setVariables(params);
        }
        String emailContent = springTemplateEngine.process(templateName, context);
        sendHtmlMail(to, subject, emailContent);
    }
}
```

5）定义测试类

```
package com.favorites.favoriteswebdemo.service;

import java.util.HashMap;
import java.util.Map;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceTest {
    
    @Resource
    private MailService mailService;

    @Test
    public void testSendTemplateMail() {
        Map<String,Object> context=new HashMap<>();
        context.put("id", "006");
        mailService.sendTemplateMail("emailTemplate", context,"yutian1012@126.com", "主题：这是模板邮件");
    }
}
```