---
title: 非对称加密
tags: [security]
---

### 密钥对

非对称加密包括公钥和私钥，是由大量的随机素数和加密方法产生的。
With public key cryptography, a user has a pair of public and private keys. These are generated using a large prime number and a key function.

![](/images/java_basic/security/250px-Public_key_making.svg.png)

### 公钥加密私钥解密

传输时使用公钥加密私钥解密

The keys are related mathematically, but cannot be derived from one another. With these keys we can encrypt messages. For example, if Bob wants to send a message to Alice, he can encrypt a message using her public key. Alice can then decrypt this message using her private key. Only Alice can decrypt this message as she is the only one with the private key.

![](/images/java_basic/security/280px-Public_key_encryption.svg.png)

### 签名

签名是一个身份识别的过程，私钥加密公钥解密，客户端就能识别服务器的合法性。

Messages can also be signed. This allows you to ensure the authenticity of the message. If Alice wants to send a message to Bob, and Bob wants to be sure that it is from Alice, Alice can sign the message using her private key. Bob can then verify that the message is from Alice by using her public key.

![](/images/280px-Public_key_signing.svg.png)