---
title: github开源项目云收藏（附件邮件内容发送）
tags: [project]
---

1）发送邮件

定义MailService接口

```
package com.favorites.favoriteswebdemo.service;

public interface MailService { 
    public void sendAttachmentsMail(String to, String subject, String content, String filePath);
}
```

2）定义MailServiceImpl实现类

```
package com.favorites.favoriteswebdemo.service;

import java.io.File;

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

@Component
public class MailServiceImpl implements MailService{
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Resource
    private JavaMailSender mailSender;

    @Value("${mail.fromMail.addr}")
    private String from;

    @Override
    public void sendAttachmentsMail(String to, String subject, String content, String filePath) {
        MimeMessage message = mailSender.createMimeMessage();
        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(content, true);
            FileSystemResource file = new FileSystemResource(new File(filePath));
            String fileName = filePath.substring(filePath.lastIndexOf(File.separator));
            helper.addAttachment(fileName, file);
            mailSender.send(message);
            logger.info("带附件的邮件已经发送。");
        } catch (MessagingException e) {
            logger.error("发送带附件的邮件时发生异常！", e);
        }
    }
}
```

注：该类使用MimeMessage构建邮件体

3）测试

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
    public void testSendAttachmentsMail() {
        String filePath="C:\\Users\\Public\\Pictures\\Sample Pictures\\Chrysanthemum.jpg";
        mailService.sendAttachmentsMail("yutian1012@126.com", "主题：带附件的邮件", "有附件，请查收！", filePath);
    }
}
```