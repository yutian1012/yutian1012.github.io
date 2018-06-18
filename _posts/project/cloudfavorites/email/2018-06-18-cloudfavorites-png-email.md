---
title: github开源项目云收藏（图片邮件内容发送）
tags: [project]
---

在发送html页面时，有时需要将图片也嵌入到html中。

1）发送邮件

定义MailService接口

```
package com.favorites.favoriteswebdemo.service;

public interface MailService { 
    public void sendInlineResourceMail(String to, String subject, String content, String rscPath, String rscId);
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
    public void sendInlineResourceMail(String to, String subject, String content, String rscPath, String rscId) {
        MimeMessage message = mailSender.createMimeMessage();
        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(content, true);
            FileSystemResource res = new FileSystemResource(new File(rscPath));
            helper.addInline(rscId, res);
            mailSender.send(message);
            logger.info("嵌入静态资源的邮件已经发送。");
        } catch (MessagingException e) {
            logger.error("发送嵌入静态资源的邮件时发生异常！", e);
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
    public void testSendInlineResourceMail() {
        String rscId = "neo006";
        String content="<html><body>这是有图片的邮件：<img src=\'cid:" + rscId + "\' ></body></html>";
        String imgPath = "C:\\Users\\Public\\Pictures\\Sample Pictures\\Chrysanthemum.jpg";
        mailService.sendInlineResourceMail("yutian1012@126.com", "主题：这是有图片的邮件", content, imgPath, rscId);
    }
}
```