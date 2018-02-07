---
title: 对称加密--AES
tags: [security]
---

参考：http://www.cnblogs.com/happyhippy/archive/2006/12/23/601353.html

### 1. 加密过程

DES的密钥长度为64位，而AES的密钥长度是128位

![](/imags/java_basic/security/aes-enc.jpg)

### 2. java API

1）密钥

AES的密钥长度必须是128位，多了少了都不行。即密码长度为16个字母或数字。

SecretKeySpec即实现了KeySpec（密钥规范），同时也实现了SecretKey，因此可以当作密钥的Key来使用

2）java实现

```
package org.ipph.web.cipher;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;

public class SymmetryAES {
    public static void main(String[] args)throws Exception{
        //原文数据
        String input="hello";

        //创建密钥并设置密码为welcomes111111（128位，16个字母或数字）
        SecretKeySpec secretKey=new SecretKeySpec("welcomes11111111".getBytes(),"AES");

        //Cipher指定AES算法
        Cipher cipher=Cipher.getInstance("AES");//实例化
        //设置加密模式和密钥key
        cipher.init(Cipher.ENCRYPT_MODE,secretKey);//初始化
        //加密调用doFinal方法
        byte[] result=cipher.doFinal(input.getBytes());//加密解密都是通过字节
        System.out.println("加密结果："+new String(result));
        
        //解密操作
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] deResult=cipher.doFinal(result);
        System.out.println("解密操作："+new String(deResult));
    }
}
```

### 3. 算法

参考：http://www.2cto.com/article/201112/113465.html