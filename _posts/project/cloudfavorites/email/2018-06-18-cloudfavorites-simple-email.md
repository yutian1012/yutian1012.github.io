---
title: github开源项目云收藏（文本邮件内容发送）
tags: [project]
---

1）发送邮件

定义MailService接口

```
package com.favorites.favoriteswebdemo.service;

public interface MailService {
    public void sendSimpleMail(String to, String subject, String content);
}
```

2）定义MailServiceImpl实现类

```
package com.favorites.favoriteswebdemo.service;

import javax.annotation.Resource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Component;

@Component
public class MailServiceImpl implements MailService{
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Resource
    private JavaMailSender mailSender;

    @Value("${mail.fromMail.addr}")
    private String from;

    @Override
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        try {
            mailSender.send(message);
            logger.info("简单邮件已经发送。");
        } catch (Exception e) {
            logger.error("发送简单邮件时发生异常！", e);
        }
    }
}
```

3）测试类

```
package com.favorites.favoriteswebdemo.service;

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
    public void sendSimpleMail() {
        mailService.sendSimpleMail("yutian1012@126.com", "邮件发送测试", "测试！！！");
    }
}
```