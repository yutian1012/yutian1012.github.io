---
title: 微信公众平台开发测试账号申请
tags: [weixin]
---

由于正式的申请比较费时，可先行申请测试账号进行开发测试，最终再使用申请的账号进行部署。

申请测试账号：

1）申请地址

http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login

登录可以使用微信扫一扫，进行授权登录。登录后会提供一个测试号管理。提供相应的appID，appsecret等信息。

2）填写接口配置信息

URL填写公网服务器IP地址或域名，Token是自己定义的。点击提交后回去验证服务是否可用。因此这里的token也是不可以随便填写的，必须与服务器上的程序相配合进行验证。

![](/images/weixin/develop/mp/testaccount.png)

3）服务器端认证代码

参考：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319

*验证URL有效性成功后即接入生效，成为开发者*

验证消息的确来自微信服务器。开发者填写URL和token信息后提交，微信服务器将发送GET请求到填写的服务器地址URL上，GET请求携带参数signature，timestamp，nonce，echostr。开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。

![](/images/weixin/develop/mp/weixin-validation.png)

```
<?php
//这里的token需要和微信公众平台的测试号中填写的token一致，因为签名时使用了该值
define("TOKEN", "mytoken");
$wechatObj = new wechatCallbackapiTest();
$wechatObj->valid();

class wechatCallbackapiTest
{
    //校验完成后需要将echostr原样返回
    public function valid()
    {
        $echoStr = $_GET["echostr"];        //随机字符串
        
        if($this->checkSignature()){
            echo $echoStr;
            exit;
        }
    }
    //1）将token、timestamp、nonce三个参数进行字典序排序 
    //2）将三个参数字符串拼接成一个字符串进行sha1加密 
    //3）开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
    private function checkSignature()
    {
        $signature = $_GET["signature"];    //微信加密签名
        $timestamp = $_GET["timestamp"];    //时间戳
        $nonce = $_GET["nonce"];            //随机数
        $token = TOKEN;
        $tmpArr = array($token, $timestamp, $nonce);
        sort($tmpArr);      //进行字典序排序
        //sha1加密后与签名对比
        if( sha1(implode($tmpArr)) == $signature ){
            return true;
        }else{
            return false;
        }
    }
}
?>
```

注：校验完成后需要将echostr原样返回，这一步主要完成双向服务器认证。

4）关注测试公众号

```
微信号:gh_2626e3fe4187 
appID:wx1552c70aa6782a58
appsecret:e6095963fb618886a32305af2d90cfe1

url:http://47.104.31.92
token:searchpatent
```

在页面下方有一个测试号二维码，扫描该二维码即关注该公众号，同时会在页面上的关注列表中出现关注的微信号记录。然后可以进一步与公众号进行交互了。

![](/images/weixin/develop/mp/weixin-focus.png)

5）综上

测试微信公众号号提供了一个扫描的二维码用来关注该公众号。同时提供了微信号，appID和appsecret，并通过了服务器url和token的认证。

成为开发者后，用户每次向公众号发送消息、或者产生自定义菜单、或产生微信支付订单等情况时，开发者填写的服务器配置URL将得到微信服务器推送过来的消息和事件，开发者可以依据自身业务逻辑进行响应，如回复消息。