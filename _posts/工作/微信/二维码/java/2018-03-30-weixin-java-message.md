---
title: 微信接收消息--java
tags: [weixin]
---

参考：http://www.cnblogs.com/jerehedu/p/6534110.html

1）微信接收消息

微信将向我们的平台发送消息时，是使用Post请求，并以XML的格式发送与接收数据。再向微信回送消息时也必须满足微信指定的xml消息格式。

*注意：微信服务器在五秒内收不到响应会断掉连接，并且重新发起请求，总共重试三次。假如服务器无法保证在五秒内处理并回复，可以直接回复空串，微信服务器不会对此作任何处理，并且不会发起重试。处理方法，先回复空字符串，处理完成后在向微信推送数据。*

2）微信发送的消息格式

```
<xml>
    <ToUserName><![CDATA[toUser]]></ToUserName>
    <FromUserName><![CDATA[fromUser]]></FromUserName>
    <CreateTime>1348831860</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[this is a test]]></Content>
    <MsgId>1234567890123456</MsgId>
</xml>
```