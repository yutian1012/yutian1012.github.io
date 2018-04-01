---
title: 微信实现校验（java）
tags: [weixin]
---

平台接入前首先需要通过微信服务器的校验，即验证第三方服务器是否可用。我们编辑的所有与微信交互的逻辑都是通过该平台实现的。

参考：https://www.cnblogs.com/jerehedu/p/6377759.html

1）创建微信的入口controller

创建包org.ipph.app.weixin.controller，在该包下创建WeixinController类。

```
package org.ipph.app.weixin.controller;

import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.ipph.app.weixin.util.WeixinValidUtil;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.http.ResponseEntity;
@Controller
@RequestMapping("/app")
public class WeixinController {
    
    @Resource
    private WeixinMessageContext weixinMessageContext;
    
    @RequestMapping(value="/index")
    @ResponseBody
    public ResponseEntity<String> ResponseEntity(HttpServletRequest request,HttpServletResponse response) throws IOException{
        response.setCharacterEncoding("UTF-8");
        HttpHeaders headers = new HttpHeaders();
        MediaType mediaType = new MediaType("text","html",Charset.forName("utf-8"));
        headers.setContentType(mediaType);
        
        return new ResponseEntity<String>(getResponseText(request),headers,HttpStatus.OK);
    }
    /**
     * 处理返回文档信息
     * @param request
     * @return
     */
    private String getResponseText(HttpServletRequest request) {
        
        String result=null;
        String echostr =request.getParameter("echostr");
        if(null!=echostr) {//校验数据
            String signature=request.getParameter("signature");
            String timestamp=request.getParameter("timestamp");
            String nonce=request.getParameter("nonce");
            if(WeixinValidUtil.valid(timestamp, nonce, signature)) {
                result=echostr;
            }
        }
        return result;
    }
    
}
```

2）创建校验类

创建org.ipph.app.weixin.util包，在该包下创建WeixinValidUtil类

```
package org.ipph.app.weixin.util;

import java.util.Arrays;

import org.ipph.app.weixin.WeixinConstant;

public class WeixinValidUtil {
    
    /**
     * 微信校验
     * @param timestamp
     * @param nonce
     * @param signature
     * @return
     */
    public static boolean valid(String timestamp,String nonce,String signature) {
        String[] params=new String[] {WeixinConstant.TOKEN,timestamp,nonce};
        Arrays.sort(params);
        StringBuilder temp=new StringBuilder();
        for(String s:params) {
            temp.append(s);
        }
        String result=EncrytUtil.encrytSHA1(temp.toString());
        return result.equals(signature);
    }
    
}
```

3）创建加密工具类

```
package org.ipph.app.weixin.util;

import java.security.MessageDigest;

public class EncrytUtil {
    
    private static final char[] HEX_DIGITS = {'0', '1', '2', '3', '4', '5',  
            '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};
    
    /**
     * 加密
     * @param str
     * @return
     */
    public static String encrytSHA1(String str) {  
        if (str == null) {  
            return null;
        }  
        try {  
            MessageDigest messageDigest = MessageDigest.getInstance("SHA1");  
            messageDigest.update(str.getBytes());  
            return getFormattedText(messageDigest.digest());  
        } catch (Exception e) {  
            throw new RuntimeException(e);  
        }  
    }
    
    /** 
    * Takes the raw bytes from the digest and formats them correct. 
    * 
    * @param bytes the raw bytes from the digest. 
    * @return the formatted bytes. 
    */  
    private static String getFormattedText(byte[] bytes) {  
        int len = bytes.length;  
        StringBuilder buf = new StringBuilder(len * 2);  
        // 把密文转换成十六进制的字符串形式  
        for (int j = 0; j < len; j++) {  
            buf.append(HEX_DIGITS[(bytes[j] >> 4) & 0x0f]);  
            buf.append(HEX_DIGITS[bytes[j] & 0x0f]);  
        }  
        return buf.toString();  
    }
}
```