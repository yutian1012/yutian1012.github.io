---
title: 微信接收响应文本消息（java）
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

3）第三方服务向微信服务器发送消息格式

文本发送消息格式与接收格式是一样的

4）修改微信服务器的入口类

修改WeixinController类，处理接收用户发送的消息，并自动回复数据。

```
@Controller
@RequestMapping("/app")
public class WeixinController {
    
    @Resource
    private WeixinMessageContext weixinMessageContext;
    
    @RequestMapping(value="/index")
    @ResponseBody
    public ResponseEntity<String> ResponseEntity(HttpServletRequest request,HttpServletResponse response) throws IOException{
        response.setCharacterEncoding("UTF-8");
        HttpHeaders headers = new HttpHeaders();
        MediaType mediaType = new MediaType("text","html",Charset.forName("utf-8"));
        headers.setContentType(mediaType);
        
        return new ResponseEntity<String>(getResponseText(request),headers,HttpStatus.OK);
    }
    /**
     * 处理返回文档信息
     * @param request
     * @return
     */
    private String getResponseText(HttpServletRequest request) {
        
        String result=null;
        String echostr =request.getParameter("echostr");
        if(null!=echostr) {//校验数据
            String signature=request.getParameter("signature");
            String timestamp=request.getParameter("timestamp");
            String nonce=request.getParameter("nonce");
            if(WeixinValidUtil.valid(timestamp, nonce, signature)) {
                result=echostr;
            }
        }else {
            InputStream in=null;
            try {
                in=request.getInputStream();
                MessageModel message=WeixinMessageUtil.getMessage(MessageModel.class, in);
                if(null!=message) {
                    return weixinMessageContext.responseMessage(message);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                if(null!=in) {
                    try {
                        in.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        return result;
    }
}
```

2）处理接收到的消息

使用JAXB方式解析xml并返回数据.

a.消息实体类

```
package org.ipph.app.weixin.model;

import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement(name="xml")
public class MessageModel {
    private String ToUserName;
    
    private String FromUserName;
    
    private Long CreateTime;
    
    private String MsgType;
    
    private String Content;
    
    private String MsgId;
    public String getToUserName() {
        return ToUserName;
    }
    @XmlElement(name="ToUserName")
    public void setToUserName(String toUserName) {
        ToUserName = toUserName;
    }
    public String getFromUserName() {
        return FromUserName;
    }
    @XmlElement(name="FromUserName")
    public void setFromUserName(String fromUserName) {
        FromUserName = fromUserName;
    }
    public Long getCreateTime() {
        return CreateTime;
    }
    @XmlElement(name="CreateTime")
    public void setCreateTime(Long createTime) {
        CreateTime = createTime;
    }
    public String getMsgType() {
        return MsgType;
    }
    @XmlElement(name="MsgType")
    public void setMsgType(String msgType) {
        MsgType = msgType;
    }
    public String getContent() {
        return Content;
    }
    @XmlElement(name="Content")
    public void setContent(String content) {
        Content = content;
    }
    public String getMsgId() {
        return MsgId;
    }
    @XmlElement(name="MsgId")
    public void setMsgId(String msgId) {
        MsgId = msgId;
    }  
}
```

b.消息格式化输出工具类

```
package org.ipph.app.weixin.util;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.StringReader;
import java.nio.charset.Charset;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.Marshaller;
import javax.xml.bind.Unmarshaller;

import org.dom4j.DocumentException;
import org.ipph.app.weixin.model.MessageModel;
import org.ipph.app.weixin.xml.MessageHandler;

public class WeixinMessageUtil {
    public static <T> T getMessage(Class<T> clazz,InputStream in) {
        BufferedReader reader=null;
        try {
            reader = new BufferedReader(new InputStreamReader(in,Charset.forName("UTF-8")));
            char[] buf=new char[1024];
            StringBuilder message=new StringBuilder();
            int len=0;
            while((len=reader.read(buf))!=-1) {
                message.append(new String(buf,0,len));
            }
            if(message.length()>0) {
                return xml2Bean(clazz, message.toString());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
    
    /**
     * xml字符串转成对应的对象
     * @param xml
     * @throws Exception
     */
    @SuppressWarnings("unchecked")
    public static <T> T xml2Bean(Class<T> clazz,String xml)throws Exception{
        
        JAXBContext context = JAXBContext.newInstance(clazz);// 获取上下文对象  
        
        Unmarshaller unmarshaller=context.createUnmarshaller();//根据上下文获取Unmarshaller对象
          
        T message=(T)unmarshaller.unmarshal(new StringReader(xml));
        //T message=(T)unmarshaller.unmarshal(new InputSource(xml));
        
        //System.out.println(message);
        return message;
    }
    
    /**
     * 将对象转换成xml字符串
     * @throws Exception
     */
    public static <T> String bean2Xml(Class<T> clazz,T bean)throws Exception{
        JAXBContext context = JAXBContext.newInstance(clazz);// 获取上下文对象  
        // 根据上下文获取marshaller对象
        Marshaller marshaller = context.createMarshaller();   
        // 设置编码字符集  
        marshaller.setProperty(Marshaller.JAXB_ENCODING, "UTF-8");  
        // 格式化XML输出，有分行和缩进
        marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);
        
        marshaller.setProperty(Marshaller.JAXB_FRAGMENT, false);
          
        //marshaller.marshal(getSimpleDepartment(),System.out);// 打印到控制台  
          
        ByteArrayOutputStream baos = new ByteArrayOutputStream();  
        marshaller.marshal(bean, baos);  
        return new String(baos.toByteArray()); // 生成XML字符串  
    }
    
}
```

3）返回消息的处理

这里使用策略模式来实现，定义了

a.枚举类

用于判断微信服务器发送过来的数据是消息还是事件。事件包括关注和取消关注。

```
package org.ipph.app.weixin.enumeration;

public enum MessageTypeEnum {
    TEXT("text"),EVENT("event");
    
    private String name;
    private MessageTypeEnum(String name) {
        this.name=name;
    }
    public String getName() {
        return this.name;
    }
}
```

b.定义消息处理枚举

其中dealContent数组中可以定义处理的关键词，即用户发送的内容与关键词匹配，则交由该枚举对应的处理类来处理。

```
package org.ipph.app.weixin.enumeration;

import org.ipph.app.weixin.message.IMessageResponse;
import org.ipph.app.weixin.message.TextMessageResponseImpl;

public enum MessageResponseEnum {
    TEXT("text",new TextMessageResponseImpl(),null);
    
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

c.策略判断入口类WeixinMessageConext

WeixinMessageConext类，根据消息的内容能够找到相应的策略实现类，具体的实现交由相应的实现类来实现。

```
package org.ipph.app.weixin.message;

import org.ipph.app.weixin.enumeration.MessageResponseEnum;
import org.ipph.app.weixin.enumeration.MessageTypeEnum;
import org.ipph.app.weixin.model.MessageModel;
import org.springframework.stereotype.Component;

@Component
public class WeixinMessageContext implements IMessageResponse{

    @Override
    public String responseMessage(MessageModel message) {
        //获取消息类型，是事件类型的消息还是文本消息
        MessageTypeEnum type=getMessageType(message);
        
        String result=null;
        
        switch (type) {
        case TEXT:
            result=responseMessageByContent(message);
            break;
        case EVENT:
            result=responseEventMessage(message);
            break;
        default:
            break;
        }
        return result;
    }
    /**
     * 返回消息，根据消息的内容能够找到相应的策略实现类，
     * @param message
     * @return
     */
    private String responseMessageByContent(MessageModel message) {
        
        String content=message.getContent();
        
        IMessageResponse messageResponse=null;
        
        MessageResponseEnum responseEnum=MessageResponseEnum.TEXT;
        boolean flag=false;
        for(MessageResponseEnum response:MessageResponseEnum.values()) {
            if(null==content||"".equals(content)) break;
            //根据关键词进行恢复
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
    /**
     * 返回事件响应消息，暂不实行事件消息的响应
     * @param message
     * @return
     */
    private String responseEventMessage(MessageModel message) {
        return null;
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
    
}
```

b.定义消息响应接口

```
package org.ipph.app.weixin.message;

import org.ipph.app.weixin.model.MessageModel;

public interface IMessageResponse {
    
    public String responseMessage(MessageModel message);
}
```

c.定义文本消息的响应内容

```
package org.ipph.app.weixin.message;

import org.ipph.app.weixin.model.MessageModel;
import org.ipph.app.weixin.util.WeixinMessageUtil;

public class TextMessageResponseImpl implements IMessageResponse{

    @Override
    public String responseMessage(MessageModel message) {
        String content=message.getContent();
        StringBuilder result=new StringBuilder();
        
        //设置返回消息
        MessageModel response=new MessageModel();
        response.setToUserName(message.getFromUserName());
        response.setFromUserName(message.getToUserName());
        if(result.length()==0) {
            result.append("您输入的数据：").append(content);
        }
        response.setContent(result.toString());
        response.setCreateTime(System.currentTimeMillis());
        response.setMsgType("text");
        
        try {
            //调用工具类型，将对象转换成字符串输出
            return WeixinMessageUtil.bean2Xml(MessageModel.class, response);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

注：返回文本消息的格式与接收消息的格式是一样的，只是将原发送者与目标接收者做一个调换，然后将要会送的消息设置到content字段中。