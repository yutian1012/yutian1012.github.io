---
title: github开源项目云收藏（忘记密码邮箱找回）
tags: [project]
---

首先系统中需要引入邮箱的配置。

1）编辑IndexController类

```
/**
 * 页面点击忘记密码连接，跳转到相应页面，填写接收验证的邮箱（用户注册时填写的邮箱）
 * @return
 */
@RequestMapping(value="/forgotPassword",method=RequestMethod.GET)
public String forgotPassword() {
    return "user/forgotpassword";
}
/**
 * 重置密码连接--跳转到相应页面（点击邮件的验证连接）
 * @param email
 * @return
 */
@RequestMapping(value="/newPassword",method=RequestMethod.GET)
public String newPassword(String email) {
    return "user/newpassword";
}
```

注：该类的两个方法用于定位到templates/user下的模板文件，填写相关信息

2）编辑UserController类

为验证连接设置一个有效期，并使用MD5加密相关的字符串，来验证链接的有效性。另外，将相关的验证信息同时写入到数据库User表中，用来验证重置密码的合法性。

```
/**
 * 发送重置密码邮件
 * @param email
 * @return
 */
@RequestMapping(value = "/sendForgotPasswordEmail", method = RequestMethod.POST)
public Response sendForgotPasswordEmail(String email) {
    try {
        User registUser = userService.findByEmail(email);
        if (null == registUser) {
            return result(ExceptionMsg.EmailNotRegister);
        }
        String secretKey = UUID.randomUUID().toString(); // 密钥
        Timestamp outDate = new Timestamp(System.currentTimeMillis() + 30 * 60 * 1000);// 30分钟后过期
        long date = outDate.getTime() / 1000 * 1000;
        userService.setOutDateAndValidataCode(outDate+"", secretKey, email);
        
        mailService.sendForgotPasswordEmail(email + "$" + date + "$" + secretKey, email);
        
    } catch (Exception e) {
        // TODO: handle exception
        logger.error("sendForgotPasswordEmail failed, ", e);
        return result(ExceptionMsg.FAILED);
    }
    return result(ExceptionMsg.SUCCESS);
}

/**
 * 忘记密码-设置新密码
 * @param newpwd
 * @param email
 * @param sid
 * @return
 */
@RequestMapping(value = "/setNewPassword", method = RequestMethod.POST)
public Response setNewPassword(String newpwd, String email, String sid) {
    try {
        User user = userService.findByEmail(email);
        Timestamp outDate = Timestamp.valueOf(user.getOutDate());
        if(outDate.getTime() <= System.currentTimeMillis()){ //表示已经过期
            return result(ExceptionMsg.LinkOutdated);
        }
        String key = user.getEmail()+"$"+outDate.getTime()/1000*1000+"$"+user.getValidataCode();//数字签名
        String digitalSignature = MD5Util.encrypt(key);
        if(!digitalSignature.equals(sid)) {
             return result(ExceptionMsg.LinkOutdated);
        }
        userService.setNewPassword(getPwd(newpwd), email);
    } catch (Exception e) {
        logger.error("setNewPassword failed, ", e);
        return result(ExceptionMsg.FAILED);
    }
    return result(ExceptionMsg.SUCCESS);
}
//获取加密的密码
protected String getPwd(String password){
    try {
        String pwd = MD5Util.encrypt(password+Const.PASSWORD_KEY);
        return pwd;
    } catch (Exception e) {
        logger.error("密码加密异常：",e);
    }
    return null;
}
```

3）邮件发送重置密码连接

在application.properties中配置相关的属性

```
##### mail forget password #####
mail.subject.forgotpassword=密码重置邮件
mail.content.forgotpassword=请点击以下地址:<br /><a href='{0}'>重置密码</a>
forgotpassword.url=http://127.0.0.1:9090/newPassword
```

a.编辑MailService接口，添加相应的接口方法

```
public void sendForgotPasswordEmail(String key,String to);
```

b.实现类MailServiceImpl

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

import com.favorites.favoriteswebdemo.util.MD5Util;
import com.favorites.favoriteswebdemo.util.MessageUtil;

@Component
public class MailServiceImpl implements MailService{
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Resource
    private JavaMailSender mailSender;
    
    @Resource
    private SpringTemplateEngine springTemplateEngine;

    @Value("${mail.fromMail.addr}")
    private String from;
    @Value("${mail.subject.forgotpassword}")
    private String mailSubject;
    @Value("${mail.content.forgotpassword}")
    private String mailContent;
    @Value("${forgotpassword.url}")
    private String forgotpasswordUrl;

    @Override
    public void sendForgotPasswordEmail(String key,String to) {
        
        String digitalSignature = MD5Util.encrypt(key);// 数字签名
        String resetPassHref = forgotpasswordUrl + "?sid="+ digitalSignature +"&email="+to;
        String emailContent = MessageUtil.getMessage(mailContent, resetPassHref);//替换连接
        
        MimeMessage mimeMessage = mailSender.createMimeMessage();   
        try {
            MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(mailSubject);
            helper.setText(emailContent, true);
            mailSender.send(mimeMessage);
        }catch (MessagingException e) {
            logger.error("邮件发送发生异常！", e);
        }
    }
}
```

4）相关工具类

a.MD5Util类

```
package com.favorites.favoriteswebdemo.util;

import java.security.MessageDigest;

public class MD5Util {
    public static String encrypt(String dataStr) {
        try {
            MessageDigest m = MessageDigest.getInstance("MD5");
            m.update(dataStr.getBytes("UTF8"));
            byte s[] = m.digest();
            String result = "";
            for (int i = 0; i < s.length; i++) {
                result += Integer.toHexString((0x000000FF & s[i]) | 0xFFFFFF00).substring(6);
            }
            return result;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }
}
```

b.MessageUtil类

```
package com.favorites.favoriteswebdemo.util;

public class MessageUtil {
    public static String getMessage(String template, String... keys) {
        int count = 0;
        for (String key : keys) {
            template = template.replace("{" + count++ + "}", key);
        }
        return template;
    }
}
```