---
title: 对称加密--DES
tags: [security]
---

加密和解密都是使用同一个密钥key.

### 1. DES算法--对bit位进行加密

Data EncryptionStandard,即数据加密标准。

字符串转字节，getBytes方法

```
String str="abc";
byte[] bytes=str.getBytes();
for(byte b:bytes){
    System.out.println(b);//输出97，98，99
}
```

1）字符与字节

utf-8编码是一个变长编码，汉字占3个字节，字母和数字占1个字节。

注：理解变长编码。

2）字节与bit位

字节是计算机中存储的数据，bit位是用来传输的。

```
System.out.println(Integer.toBinaryString(1));可以输出对应的bit位
```

3）原理

将明文分成一个个64bit位的大小（即每64bit一组），对这每个64个bit位进行置换，位移等复杂操作进行加密，最后将这些加密后的数据整合到一起形成密文。

原理：将64bit位数据进行各种打乱操作，形成密文。解密时由于知道如何打乱的，再还原回去就可以了。

### 2. java API

1）帮助文档

加密的类在javax包下（即扩展库中），具体包名为：javax.crypto，核心类是Cipher。

注：以java开头的包名提供的是java基础类库，以javax开头的包名提供的是java的扩展库（extension）。

2）Cipher类

加密解密都是通过这个类来实现，Cipher.getInstance获取一个实例，需要传递一个字符串参数，该参数表示采用的加密算法（如DES算法）

3）java实现

DES加密算法的密码必须是64bit位，即8个字母或数字。

```
package org.ipph.web.cipher;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;

/**
 * 对称加密
 */
public class SymmetryDES {
    public static void main(String[] args)throws Exception {
        //原文数据
        String input="hello";

        //密钥工厂
        SecretKeyFactory keyFactory=SecretKeyFactory.getInstance("DES");
        //密钥规格，ctrl+T查看类继承结构
        DESKeySpec keySpec=new DESKeySpec("welcomes".getBytes());//设置密码
        //创建密钥，接口为Key
        SecretKey secretKey=keyFactory.generateSecret(keySpec);

        //Cipher指定DES算法
        Cipher cipher=Cipher.getInstance("DES");//实例化
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

注：加密操作还是解密操作只是通过init方法指定模式来决定，最终加密和解密操作都是通过doFinal方法实现的。

4）DES算法的工作和填充模式

构造Cipher对象时可以指定相应的工作模式和填充模式

```
Cipher cipher=Cipher.getInstance("DES/ECB/PKCS5Padding");//实例化
```

注：注：Cipher初始化如果只使用”DES”作为参数，默认使用的是DES/ECB/PKCS5Padding字符串。

（1）工作模式

DES/ECB/PKCS5Padding表示：算法/工作模式/填充模式

ECB工作模式：表示电子密码本，每块（64位为一块）独立加密。

CBC工作模式：密码分组链接，每块加密依赖于前一块的密文。

注：ECB可以并行加密，效率高，CBC需要串行加密，加密强度高。

（2）填充模式

NoPadding模式：不填充，但要求待加密的明文必须是8的整数倍（需要凑成64位明文进行加密）。如果不是会抛出异常。

PKCS5Padding模式：当输入的明文不是8的整数倍时，会自动填充。
