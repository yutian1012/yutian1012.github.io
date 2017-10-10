---
title: cas服务器跨域问题
tags: [cas]
---

参考：http://blog.csdn.net/pucao_cug/article/details/70216109

跨域主要问题是浏览器cookie信息不能共享的问题。

解决方式主要是通过重定向方式。ticket的信息是通过url传递参数的方式向不同系统中发送的。

认证过程：

1）A系统登录时，会重定向到cas服务器。

2）cas服务器登录后将ticket等信息保存在浏览器的cookie中。然后重定向回A系统，由于存在跨域的问题，A服务器不能获取到cas服务器存储在浏览器中的cookie信息，即无法获取ticket。因此，在重定向A服务器时会在url参数上追加上ticket的参数值。

3）A系统接收到cas的重定向url，A系统处理请求，能够从url中获取ticket信息，使用ticket到cas端再次进行一个认证，认证这个ticket是否有效，并能从cas服务器端获取登录的用户名，进而实现A系统的登录，即A系统登录成功。

4）B系统登录，不能直接访问cas存放到浏览器中的cookie信息，也会重定向到cas服务器。在重定向到Cas服务器时，会从浏览器cookie中获取保存的ticket信息（这里不存在跨域问题），这个重定向到操作等价于在浏览器中直接输入cas服务器地址，会将cas服务器保持的cookie也一并发送到cas服务器。

5）cas服务器从请求中能够获取cookie中的ticket，发现是认证通过的ticket，则重定向到B系统，并在url后追加上ticket参数值。B系统接收到cas的重定向请求，认证过程与A系统一样，从而实现B系统的登录。

![](/images/work/cas/cas-cross-domain.png)