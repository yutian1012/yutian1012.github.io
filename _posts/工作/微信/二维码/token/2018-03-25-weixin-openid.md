---
title: 微信获取openid
tags: [weixin]
---

openId是指关注公众号的微信用户的ID信息，而公众号本身并没有openID属性。

1）什么是openid

一个微信号与一个公众号对应一个固定不变的openid，所以一个微信号在一个公众号下的openid是不变的。

在关注者与公众号产生消息交互后，公众号可获得关注者的OpenID（加密后的微信号，每个用户对每个公众号的OpenID是唯一的。对于不同公众号，同一用户的openid不同）。公众号可通过本接口来根据OpenID获取用户基本信息，包括昵称、头像、性别、所在城市、语言和关注时间。

注：openID是一个被动获取的信息。是先有用户向服务发起，然后服务才能获取对应微信用户的openid。

2）如何获取openid

在关注者与公众号产生消息交互后，公众号可获得关注者的OpenID。

openid的获取，当微信公众号关注用户向公众号发生消息时，通过查看apache的访问日志，可以看到传递过来的openid信息。

```
# 查看apache的访问日志
tail -f /usr/local/apache/logs/access_log

# 输出信息
POST /?signature=4402d9d81cd757dedaxxx4b14a3627a3248e69fddc0&timestamp=1521975961&nonce=1104944602&openid=o5fX40UIKj6v-xxxcGBxh82r7Ju74AQ HTTP/1.1" 200 287
```

3）监听关注者发来的消息，并进行回送消息

当普通微信用户向公众账号发消息时，微信服务器将POST消息的XML数据包到开发者填写的URL上，即开发者的服务器上。

流程过程：

用户发送消息会先发送到微信服务器上，微信服务器根据用户发送消息的参数信息，定位到微信公众号。通过公众号的信息获取设置的开发者服务器地址，然后将用户发送的信息再转发给开发者服务器。开发者服务器就会收到用户发来的消息。

开发者服务器通过解析用户发来的数据，能够确定用户的openid信息，进而将信息再会送给用户。回送过程也是先将信息response给微信服务器，微信服务器再传给用户。

4）apache服务器查看日志信息

有时在服务器运时，没有任何的返回信息，可以在apache日志中查看相关的服务错误信息。进而定位错误，修改程序。

```
tail /usr/local/apache/logs/error_log
```

5）错误信息

参考：http://php.net/always-populate-raw-post-data

php日志中提示“Notice:  Undefined index: HTTP_RAW_POST_DATA in xxx”

```
# 替换代码
$postStr = $GLOBALS["HTTP_RAW_POST_DATA"];
# 替换为下面的代码
$postStr = isset($GLOBALS['HTTP_RAW_POST_DATA']) ? $GLOBALS['HTTP_RAW_POST_DATA'] : file_get_contents("php://input"); 
```

官方解释

```
The preferred method for accessing raw POST data is php://input, and $HTTP_RAW_POST_DATA is deprecated in PHP 5.6.0 onwards.
```