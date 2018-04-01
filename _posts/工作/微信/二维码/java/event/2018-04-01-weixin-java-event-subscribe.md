---
title: 微信实现关注事件的处理（java）
tags: [weixin]
---

1）事件推送格式

```
<xml>
    <ToUserName>< ![CDATA[toUser] ]></ToUserName>
    <FromUserName>< ![CDATA[FromUser] ]></FromUserName>
    <CreateTime>123456789</CreateTime>
    <MsgType>< ![CDATA[event] ]></MsgType>
    <Event>< ![CDATA[subscribe] ]></Event>
</xml>
```

具体xml格式还需要参考微信官方提供的文档。

2）事件回送消息格式

这里可以简单的向关注的用户发送文本消息作为回送消息。

```
<xml>
    <ToUserName><![CDATA[toUser]]></ToUserName>
    <FromUserName><![CDATA[fromUser]]></FromUserName>
    <CreateTime>1348831860</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[this is a test]]></Content>
</xml>
```

3）消息处理

这里屏蔽了具体事件，只需要针对相应的事件向content字段中注入相关的关键字。具体的消息响应方式只需要在具体的实现类中获取。

a.上下文出类

设置content值为subscribe

```
@Component
public class WeixinMessageContext implements IMessageResponse{

    @Override
    public String responseMessage(MessageModel message) {
        MessageTypeEnum type=getMessageType(message);
        
        switch (type) {
        case TEXT://文本消息类型
            
            break;
        case IMAGE://图文消息类型
            
            break;
        case EVENT://事件消息类型
            return responseEventMessage(message);
            
        default:
            break;
        }
        return getResponseMessage(message);
    }
    
    /**
     * 返回事件响应消息
     * @param message
     * @return
     */
    private String responseEventMessage(MessageModel message) {
        EventTypeEnum type=getEventType(message);
        
        switch (type) {
        case SUBSCRIBE://关注事件
            message.setContent("subscribe");//设置处理关键字
            break;
        case UNSUBSCRIBE://取消关注事件
            
            break;

        default:
            break;
        }
        return getResponseMessage(message);
    }
    /**
     * 获取消息类型
     * @param message
     * @return
     */
    private MessageTypeEnum getMessageType(MessageModel message) {
        String type=message.getMsgType();
        if(null==type||"".equals(type)) return null;
        
        return MessageTypeEnum.valueOf(type.toUpperCase());
    }
    
    /**
     * 获取事件类型
     * @param message
     * @return
     */
    private EventTypeEnum getEventType(MessageModel message) {
        String type=message.getEvent();
        if(null==type||"".equals(type)) return null;
        
        return EventTypeEnum.valueOf(type.toUpperCase());
    }
    
    /**
     * 返回消息
     * @param message
     * @return
     */
    private String getResponseMessage(MessageModel message) {
        
        String content=message.getContent();
        
        IMessageResponse messageResponse=null;
        
        MessageResponseEnum responseEnum=MessageResponseEnum.TEXT;
        boolean flag=false;
        for(MessageResponseEnum response:MessageResponseEnum.values()) {
            if(null==content||"".equals(content)) break;
            
            String[] dealContent=response.getDealContent();
            if(null!=dealContent) {
                for(String dealText:dealContent) {
                    if(content.equals(dealText)) {
                        responseEnum=response;
                        flag=true;
                        break;
                    }
                }
            }
            
            if(flag) break;
        }
        
        if(null!=responseEnum) {
            messageResponse=responseEnum.getMessageResponse();
        }
        
        if(null!=messageResponse) {
            return messageResponse.responseMessage(message);
        }
        
        return null;
    }
    
}
```

b.响应的事件类型枚举类

```
package org.ipph.app.weixin.enumeration;

public enum EventTypeEnum {
    LOCATION("location"),SUBSCRIBE("subscribe"),UNSUBSCRIBE("unsubscribe"),CLICK("click"),VIEW("view"),SCAN("scan");
    
    private String name;
    private EventTypeEnum(String name) {
        this.name=name;
    }
    public String getName() {
        return this.name;
    }
}
```

c.消息类型枚举类

```
package org.ipph.app.weixin.enumeration;

public enum MessageTypeEnum {
    TEXT("text"),IMAGE("image"),VOICE("voice"),VIDEO("video"),SHORTVIDEO("shortVideo"),LINK("link"),
    EVENT("event");
    
    private String name;
    private MessageTypeEnum(String name) {
        this.name=name;
    }
    public String getName() {
        return this.name;
    }
}
```

d.消息处理枚举适配

增加了SubscribeMessageResponseImpl类，用来处理注册事件。

```
package org.ipph.app.weixin.enumeration;

import org.ipph.app.weixin.message.IMessageResponse;
import org.ipph.app.weixin.message.TextMessageResponseImpl;
import org.ipph.app.weixin.message.SubscribeMessageResponseImpl;

public enum MessageResponseEnum {
    TEXT("text",new TextMessageResponseImpl(),null),
    SUSCRIBE("subscribe",new SubscribeMessageResponseImpl(),new String[] {"subscribe"});
    
    private String name;
    private IMessageResponse messageResponseImpl=null;
    private String[] dealContent=null;
    private MessageResponseEnum(String name,IMessageResponse messageResponse,String[] dealContent) {
        this.name=name;
        this.messageResponseImpl=messageResponse;
        this.dealContent=dealContent;
    }
    public String getName() {
        return this.name;
    }
    public IMessageResponse getMessageResponse() {
        return this.messageResponseImpl;
    }
    public String[] getDealContent() {
        return this.dealContent;
    }
}
```

4）事件处理类

a.具体实现类

```
package org.ipph.app.weixin.message;

import org.ipph.app.weixin.model.MessageModel;

public class SubscribeMessageResponseImpl extends AbstractMessageResponse{

    @Override
    public String setTextMessageContent(MessageModel message) {
        StringBuilder result=new StringBuilder();
        result.append("欢迎关注专利扫一扫，通过输入申请号，即可查询专利的著录项信息！");
        return result.toString();
    }
}
```

b.抽象模板类

```
package org.ipph.app.weixin.message;

import org.ipph.app.weixin.model.MessageModel;
import org.ipph.app.weixin.util.WeixinMessageUtil;
/**
 * 模板类
 */
public abstract class AbstractMessageResponse implements IMessageResponse{
    @Override
    public String responseMessage(MessageModel message) {
        MessageModel response=new MessageModel();
        response.setToUserName(message.getFromUserName());
        response.setFromUserName(message.getToUserName());
        response.setCreateTime(System.currentTimeMillis());
        response.setMsgType(getMsgType());
        
        //调用模板方法获取响应数据
        response.setContent(setTextMessageContent(message));
        
        try {
            return WeixinMessageUtil.bean2Xml(MessageModel.class, response);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
    /**
     * 获取响应数据
     * @param message
     * @return
     */
    public abstract String setTextMessageContent(MessageModel message);
    /**
     * 设置响应类型
     * @return
     */
    protected String getMsgType() {
        return "text";
    }
}
```

c.接口类

```
package org.ipph.app.weixin.message;

import org.ipph.app.weixin.model.MessageModel;

public interface IMessageResponse {
    public String responseMessage(MessageModel message);
}
```