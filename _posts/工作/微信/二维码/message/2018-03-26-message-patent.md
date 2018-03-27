---
title: 微信公众开发平台查询获取专利
tags: [weixin]
---

```
<?php

    $patentUrl="http://api.souips.com:8080/ipms/pat";
    //$url="http://47.104.177.116";
    define("keyParamName", "keyStr");
    define("passParamName", "valueStr");
    define("key", "com.xxyy.zhuanlitongtest");

    $result=curl_get_patent($patentUrl);

    echo "result:".$result;

    function curl_get_patent($url){
        header("Content-type: text/html; charset=utf-8");

        $post_fields=requestParams();

        $ch=curl_init();

        curl_setopt($ch,CURLOPT_URL,$url);
        curl_setopt($ch, CURLOPT_POST, 1);//post提交方式
        curl_setopt($ch, CURLOPT_POSTFIELDS, $post_fields); 

        var_dump($post_fields);

        
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_TIMEOUT, 30);

        $result=curl_exec($ch);
        //获取响应值
        $code=curl_getinfo($ch,CURLINFO_HTTP_CODE);

        echo "response code:".$code;

        if($code=='200' && $result){
            return $result;
        }
        curl_close($ch);
    }
    
    //请求参数
    function requestParams(){
        $post_fields = array(); 

        $post_fields['displayCols'] ="";//'appNumber,appDate'; //urlencode('appNumber,appDate');   
        $post_fields['dbs'] = 'FMZL';  
        $post_fields['exp'] ="申请号=('CN201210105520.X')";// urlencode("申请号=('CN201210105520.X')");   
        $post_fields['from'] = "0";   
        $post_fields['to'] = "1";
        $post_fields['order'] ="";//urlencode('-公开（公告）日');

        //加密串参数
        $timestamp=time()."000";
        $post_fields[keyParamName]=$timestamp;//末尾3位为毫秒数
        $post_fields[passParamName]=md5($timestamp.key);
        return $post_fields;
    }

?>
```