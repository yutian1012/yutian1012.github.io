---
title: 微信公众开发平台--消息处理
tags: [weixin]
---

从第三方平台的角度来区分，消息分为接收消息和发送消息。对应的消息格式是存在少许区别的。可通过官方提供的文档来查看并设置。

另外，针对事件类型的消息只会是微信服务器发送到第三方平台，不存在第三方平台向微信服务器推送事件类型的消息。即事件类型的消息是单向的，有微信服务器推送到第三方平台。

### 微信服务器发送到第三方平台的消息

微信公众平台支持的消息类型以及对应的事件关键字，微信可接收的消息类型以及可触发的事件类型有很多种，而每种类型都对应着详细的XML包，在微信公众平台开发文档中有详细解析。

用户在关注公众号后可以向公众号中发送语音，文字，图片，定位，甚至视频。响应的微信服务器会将这些信息以消息的形式发送给第三方平台。

![](/images/weixin/develop/mp/weixin-message.png)

1）可接收消息类型：

a.文本消息——text，

```
<xml>  
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>  
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>  
    <CreateTime>1348831860</CreateTime>  
    <MsgType>< ![CDATA[text] ]></MsgType>  
    <Content>< ![CDATA[this is a test] ]></Content>  
    <MsgId>1234567890123456</MsgId>  
</xml>
```

b.语音消息——voice，

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>1357290913</CreateTime>
    <MsgType>< ![CDATA[voice] ]></MsgType>
    <MediaId>< ![CDATA[media_id] ]></MediaId>
    <Format>< ![CDATA[Format] ]></Format>
    <MsgId>1234567890123456</MsgId>
</xml>
```

注：开启语言识别功能后，会将识别的语言信息存储在Recognition字段中。

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>1357290913</CreateTime>
    <MsgType>< ![CDATA[voice] ]></MsgType>
    <MediaId>< ![CDATA[media_id] ]></MediaId>
    <Format>< ![CDATA[Format] ]></Format>
    <Recognition>< ![CDATA[腾讯微信团队] ]></Recognition>
    <MsgId>1234567890123456</MsgId>
</xml>
```

注：这里的MediaId，可以调用多媒体文件下载接口拉取数据。

c.图片消息——image，

```
<xml> 
    <ToUserName>< ![CDATA[toUser] ]></ToUserName> 
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName> 
    <CreateTime>1348831860</CreateTime> 
    <MsgType>< ![CDATA[image] ]></MsgType> 
    <PicUrl>< ![CDATA[this is a url] ]></PicUrl> 
    <MediaId>< ![CDATA[media_id] ]></MediaId> 
    <MsgId>1234567890123456</MsgId> 
</xml>
```

d.视频消息——video，

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>1357290913</CreateTime>
    <MsgType>< ![CDATA[video] ]></MsgType>
    <MediaId>< ![CDATA[media_id] ]></MediaId>
    <ThumbMediaId>< ![CDATA[thumb_media_id] ]></ThumbMediaId>
    <MsgId>1234567890123456</MsgId>
</xml>
```

e.小视频消息——shortvideo

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>1357290913</CreateTime>
    <MsgType>< ![CDATA[shortvideo] ]></MsgType>
    <MediaId>< ![CDATA[media_id] ]></MediaId>
    <ThumbMediaId>< ![CDATA[thumb_media_id] ]></ThumbMediaId>
    <MsgId>1234567890123456</MsgId>
</xml>
```

f.链接消息——link

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>1351776360</CreateTime>
    <MsgType>< ![CDATA[link] ]></MsgType>
    <Title>< ![CDATA[公众平台官网链接] ]></Title>
    <Description>< ![CDATA[公众平台官网链接] ]></Description>
    <Url>< ![CDATA[url] ]></Url>
    <MsgId>1234567890123456</MsgId>
</xml>
```

g.位置消息——location，

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>1351776360</CreateTime>
    <MsgType>< ![CDATA[location] ]></MsgType>
    <Location_X>23.134521</Location_X>
    <Location_Y>113.358803</Location_Y>
    <Scale>20</Scale>
    <Label>< ![CDATA[位置信息] ]></Label>
    <MsgId>1234567890123456</MsgId>
</xml>
```

2）支持的事件推送——event

a.关注——subscribe，

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[FromUser] ]></FromUserName>
    <CreateTime>123456789</CreateTime>
    <MsgType>< ![CDATA[event] ]></MsgType>
    <Event>< ![CDATA[subscribe] ]></Event>
</xml>
```

b.取消关注——unsubscribe，

取消关注的格式与关注xml格式相同，只是Event字段的值不同，为unsubscribe。

c.上传地理位置——location

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[fromUser] ]></FromUserName>
    <CreateTime>123456789</CreateTime>
    <MsgType>< ![CDATA[event] ]></MsgType>
    <Event>< ![CDATA[LOCATION] ]></Event>
    <Latitude>23.137466</Latitude>
    <Longitude>113.352425</Longitude>
    <Precision>119.385040</Precision>
</xml>
```

d.菜单点击：

点击菜单获取消息时触发click；点击菜单跳转链接时触发view

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[FromUser] ]></FromUserName>
    <CreateTime>123456789</CreateTime>
    <MsgType>< ![CDATA[event] ]></MsgType>
    <Event>< ![CDATA[CLICK] ]></Event>
    <EventKey>< ![CDATA[EVENTKEY] ]></EventKey>
</xml>
```

e.扫描带参数二维码：

未关注时触发subscribe；

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[FromUser] ]></FromUserName>
    <CreateTime>123456789</CreateTime>
    <MsgType>< ![CDATA[event] ]></MsgType>
    <Event>< ![CDATA[subscribe] ]></Event>
    <EventKey>< ![CDATA[qrscene_123123] ]></EventKey>
    <Ticket>< ![CDATA[TICKET] ]></Ticket>
</xml>
```

注：事件KEY值，qrscene_为前缀，后面为二维码的参数值。二维码的ticket，可用来换取二维码图片，调用相关的接口即可。

已关注时触发scan

```
<xml> 
    <ToUserName>< ![CDATA[toUser] ]></ToUserName> 
    <FromUserName>< ![CDATA[FromUser] ]></FromUserName> 
    <CreateTime>123456789</CreateTime> 
    <MsgType>< ![CDATA[event] ]></MsgType> 
    <Event>< ![CDATA[SCAN] ]></Event> 
    <EventKey>< ![CDATA[SCENE_VALUE] ]></EventKey> 
    <Ticket>< ![CDATA[TICKET] ]></Ticket> 
</xml>
```

注：事件KEY值，是一个32位无符号整数，即创建二维码时的二维码scene_id

### 第三方平台响应服务器回传消息

严格来说，发送被动响应消息其实并不是一种接口，而是对微信服务器发过来消息的一次回复。这里只是作为被动的消息回复。

1）回复消息类型

a.回复文本消息

b.回复图片消息

c.回复语音消息

d.回复视频消息

e.回复音乐消息

f.回复图文消息