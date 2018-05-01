---
title: java证书库
tags: [net]
---

参考：https://www.cnblogs.com/vogueboy/p/5823178.html

参考2：https://blog.csdn.net/liaomin416100569/article/details/76020675

1）加密算法

为了网络通讯中的报文安全，一般需要对报文进行加密，目前常用的加密算法有：

a.非对称加密算法：

非对称加密算法又称公钥加密算法，包括：RSA、DSA/DSS，最常用的就是RSA算法（算法公开，可自行百度了解算法细节），算法产生一个公钥一个私钥，用公钥加密的报文只能用私钥解密，用私钥加密的报文只能用公钥解密。

b.对称加密算法：

对称加密算法包括：3DES、AES、RC4，加密密钥与解密密钥相同，一般用于只有通讯双方知道密钥的通讯方式；

c.HASH算法：

HASH算法包括：MD5、SHA1、SHA256，由哈希算法计划得到哈希值，加密过程不可逆，由哈希值不能得到原明文，一般用于作摘要签名；

2）数字证书

数字证书是由CA（Certificate Authority）机构，发行的用于网络通讯中验证身份的一种方式。数字证书中一般包含了此证书拥有者、证书使用者、证书名称、证书公钥等信息。

3）证书生成

用JDK提供的证书管理工具keytool可以制作证书，命令如下：

```
keytool -genkey -keyalg RSA -keysize 2048 -validity 36500 -alias SEC_TEST -keypass 123456 -keystore test.keystore -storepass 123456 -dname "CN=localhost,OU=DEP,O=CN,L=BJ,ST=BJ,C=CN"
```

其中，-keyalg指定算法，-keysize指定密钥大小，-validity指定有效期，单位为天；-alias别名；-keypass指定私钥使用密码；-keystore指定密钥库名称；-storepass证书库的使用密码，从里面提取公钥时需要密码；-dnameCN拥有者名字，一般为网站名或IP+端口，如www.baidu.com，OU组织机构名，O组织名，L城市，ST州或省，C国家代码。

4）密钥库的概念：

java默认的密钥库为JKS

运行上述命令，会当前目录下产生一个keystore文件（即密钥库），里面保存着密钥和证书信息；同一个秘钥库中可以存储多个证书，秘钥库必须设置一个访问的口令防止被盗（这里使用-storepass参数指定）。

注：当然也可以向当前密钥库中生成新的密钥证书。

5）查看密钥库的证书列表

```
keytool -list -keystore test.keystore -storepass 123456
```

![](/images/web/https/keytool-list.png)

6）服务器端生成公钥

```
keytool -export -alias SEC_TEST -file test_pub_cer.cer -keystore test.keystore -storepass 123456
```

注：在当前目录下会产生一个test_pub_cer.cer的证书，包含了公钥信息及证书相关信息

7）客户端端导入公钥

通讯双方假设为A和B，A发布了自己的证书并公开了公钥，B使用A的公钥加密报文并发送给A，A可以正确解密（使用A的私钥），如果A给B发送报文时，A用私钥加密，B可以用公钥解密，但这里有一个问题就是公钥是公开的，A发送给B的报文，任何有A公钥的人都可以解密，不能保证A向B发送信息的安全性。所以B也需要制作自己的证书，并对A公开自己的公钥，这样A向B发送信息里用B的公钥加密，这样B就可以用私钥解密，而其他截获信息的人因为没有私钥也不能解密；A需要将B的公钥导入自己的证书库；同时B也需要将A的公钥导入到自己的证书库中。

通信过程

```
            使用B的公钥加密消息
A端 ------------------------------> B端
（持有B的公钥）                    （使用B的私钥解密）

            使用A的公钥加密消息
A端 <------------------------------ B端
（使用A的私钥解密）                 （持有A的公钥）
```

注：A端与B端是对等的，不区分客户端和服务器。消息传送时都是使用公钥解密，确保不会被第三方劫持者不会破译（因为私钥是独有的）。

导入命令：

```
keytool -import  -file B.cer -keystore test.keystore -storepass 123456
```

提示是否信任这个认证，y,回车后即可导入。

8）https协议

https协议的实现上与上述不同，是通过双方协商一个临时的对称密钥，使用对称密码进行加密传送消息的。