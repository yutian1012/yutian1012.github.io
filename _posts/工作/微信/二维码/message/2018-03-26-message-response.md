---
title: 微信公众开发平台相应消息
tags: [weixin]
---

1）服务器端接收关注用户发送的消息

```
public function responseMsg()
{
    $postStr = isset($GLOBALS['HTTP_RAW_POST_DATA']) ? $GLOBALS['HTTP_RAW_POST_DATA'] : file_get_contents("php://input");

    if (!empty($postStr)){
        //解析字符串为SimpleXMLElement对象
        $postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
        //获取MsgType类型
        $RX_TYPE = trim($postObj->MsgType);

        switch ($RX_TYPE)
        {
            case "event"://事件的处理
                //....
                break;
            case "text":
                receiveText($postObj);
                break;
        }
        $this->logger("T ".$result);
        echo $result;
    }else {
        echo "";
        exit;
    }
}

//接收文本消息
private function receiveText($object)
{
    $keyword = trim($object->Content);
    $content = date("Y-m-d H:i:s",time());
    
    $result = $this->transmitText($object, $content);
    
    return $result;
}
```