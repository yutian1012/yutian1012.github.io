---
title: 加密工具类
tags: [security]
---

### 实现两种类型的加密

1）对称加密，能加密也能解密

2）不可逆，md5和SHA加密，一般进行消息摘要（完整性校验）

### 程序代码

```
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

import javax.crypto.Cipher;   
import javax.crypto.SecretKey;   
import javax.crypto.SecretKeyFactory;   
import javax.crypto.spec.DESKeySpec;   
import javax.crypto.spec.IvParameterSpec; 
public class EncryptUtil {

    /**
     * 使用MD5加密
     * @param inStr
     * @return
     * @throws Exception
     */
    public static String encryptMd5(String inStr) throws Exception {
        MessageDigest md = null;
        try {
            md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(inStr.getBytes());
            return new String( Base64.encodeBase64(digest));
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            throw e;
        }        
    }
       
    /**
     * 输出明文按sha-256加密后的密文
     * @param inputStr 明文
     * @return
     */
    public static synchronized String encryptSha256(String inputStr) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte digest[] = md.digest(inputStr.getBytes("UTF-8"));
            return new String(Base64.encodeBase64(digest));
        } catch (Exception e) {
            return null;
        }
    }

    /**
     * 密钥
     */
    private static final String key = "@#$%^6a7";   
           
    /**
     * 对称解密算法
     * @param message
     * @return
     * @throws Exception
     */
    public static String decrypt(String message) throws Exception {   
        byte[] bytesrc = message.getBytes("UTF-8");  
        Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");  
        //设置密钥 
        DESKeySpec desKeySpec = new DESKeySpec(key.getBytes("UTF-8"));  

        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");   
        SecretKey secretKey = keyFactory.generateSecret(desKeySpec);   
        IvParameterSpec iv = new IvParameterSpec(key.getBytes("UTF-8"));   
  
        cipher.init(Cipher.DECRYPT_MODE, secretKey, iv); 
  
        byte[] retByte = cipher.doFinal(bytesrc);   
        return new String(retByte, "UTF-8");   
    }   
      
    /**
     * 对称加密算法
     * @param message
     * @return
     * @throws Exception
     */
    public static  String encrypt(String message) throws Exception {   
        Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");   
  
        DESKeySpec desKeySpec = new DESKeySpec(key.getBytes("UTF-8"));   
  
        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");   
        SecretKey secretKey = keyFactory.generateSecret(desKeySpec);   
        IvParameterSpec iv = new IvParameterSpec(key.getBytes("UTF-8"));   
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, iv);   
  
        String str=bytesToString( cipher.doFinal(message.getBytes("UTF-8")));
        return str;
    }   
  
    /**
     * String转Byte数组--解密时，将16进制字符串转换成加密后的字节数组 
     * @param temp
     * @return
     */
    private static byte[] stringToBytes(String temp) {   
        byte digest[] = new byte[temp.length() / 2];   
        for (int i = 0; i < digest.length; i++) {   
            String byteString = temp.substring(2 * i, 2 * i + 2);   
            int byteValue = Integer.parseInt(byteString, 16);   
            digest[i] = (byte) byteValue;   
        }   
  
        return digest;   
    }   
       
    /**
     * Byte数组转String--加密后，将加密后的字节数组输出成16进制字符串
     * @param b
     * @return
     */
    private static String bytesToString(byte b[]) {   
        StringBuffer hexString = new StringBuffer();   
        for (int i = 0; i < b.length; i++) {   
            String plainText = Integer.toHexString(0xff & b[i]);   
            if (plainText.length() < 2)   
                plainText = "0" + plainText;   
            hexString.append(plainText);   
        }   
        return hexString.toString();   
    }   
   
}
```

### 加密的参数问题

1）IvParameterSpec类

```
IvParameterSpec iv = new IvParameterSpec(key.getBytes("UTF-8"));
```

注：此类指定一个初始化向量 (IV)。使用 IV 的例子是反馈模式中的密码，如，CBC 模式中的 DES。

2）Base64

引入jar包，Base64是网络上最常见的用于传输8Bit字节代码的编码方式之一。可用于在HTTP环境下传递较长的标识信息。

```
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.10</version>
</dependency>
```