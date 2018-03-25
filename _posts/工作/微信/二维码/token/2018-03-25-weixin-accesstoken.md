---
title: 微信获取accesstoken
tags: [weixin]
---

access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。开发者需要进行妥善保存。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。

### access_token

1）统一器获取路径

建议公众号开发者使用中控服务器统一获取和刷新Access_token，其他业务逻辑服务器所使用的access_token均来自于该中控服务器，不应该各自去刷新，否则容易造成冲突，导致access_token覆盖而影响业务；

2）新旧token同时可用

目前Access_token的有效期通过返回的expire_in来传达，目前是7200秒之内的值。中控服务器需要根据这个有效时间提前去刷新新access_token。在刷新过程中，中控服务器可对外继续输出的老access_token，此时公众平台后台会保证在5分钟内，新老access_token都可用，这保证了第三方业务的平滑过渡；

3）提供独立的刷新机制

Access_token的有效时间可能会在未来有调整，所以中控服务器不仅需要内部定时主动刷新，还需要提供被动刷新access_token的接口，这样便于业务服务器在API调用获知access_token已超时的情况下，可以触发access_token的刷新流程。

### 获取方式

公众号可以使用AppID和AppSecret调用本接口来获取access_token。

1）直接使用get方式获取

```
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET
```

注：grant_type值是固定的client_credential。

2）代码方式获取

```
<?php
class TokenUtil {
    //获取access_token并保存到token.txt文件中
    public static function build_access_token(){
        $ch = curl_init(); //初始化一个CURL对象
        curl_setopt($ch, CURLOPT_URL, "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=wx2e9f8435ebdb2856&secret=288db114f02b2b5cdc249ca75a4bf1cc");//设置你所需要抓取的URL
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);//设置curl参数，要求结果是否输出到屏幕上，为true的时候是不返回到网页中,假设上面的0换成1的话，那么接下来的$data就需要echo一下。
        $data = json_decode(curl_exec($ch));
        if($data->access_token){
            $token_file = fopen("token.txt","w") or die("Unable to open file!");//打开token.txt文件，没有会新建
            fwrite($token_file,$data->access_token);//重写tken.txt全部内容
            fclose($token_file);//关闭文件流
        }else{
            echo $data->errmsg;
        }
        curl_close($ch);
    }
    
    //设置定时器，每两小时执行一次build_access_token()函数获取一次access_token
    public static function set_interval(){
        ignore_user_abort();//关闭浏览器仍然执行
        set_time_limit(0);//让程序一直执行下去
        $interval = 7200;//每隔一定时间运行
        do{
            build_access_token();
            sleep($interval);//等待时间，进行下一次操作。
        }while(true);
    }
    
    //读取token
    public static function read_token(){
        $token_file = fopen("token.txt", "r") or die("Unable to open file!");
        $rs = fgets($token_file);
        fclose($token_file);
        return $rs;
    }
}
?>
```

3）页面测试工具获取

测试页面地址：https://mp.weixin.qq.com/debug/

![](/images/weixin/develop/mp/weixin-accesstoken.png)
