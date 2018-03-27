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

```
public static boolean isConnected() {
        boolean flag=false;
        //访问服务器的一个地址，看能不能正常获取链接
        if(null==PatentInterfaceConnection.INTERFACEURL){
            PatentInterfaceConnection.INTERFACEURL="http://api.souips.com:8080/ipms/pat";//"http://59.151.99.154:8080/ipms/pat";
        }
        if(null==PatentInterfaceConnection.INTERFACEPRS){
            PatentInterfaceConnection.INTERFACEPRS="http://api.souips.com:8080/ipms/prs";//"http://59.151.99.154:8080/ipms/prs";
        }
        
        DefaultHttpClient httpclient = new DefaultHttpClient();  
        HttpPost httppost = new HttpPost(PatentInterfaceConnection.INTERFACEURL);  
        httppost.setHeader("ContentType", PatentInterfaceConnection._jsonApplication);  
        //封装请求参数
        List<NameValuePair> nvps=new ArrayList<NameValuePair>();
        nvps.add(new BasicNameValuePair("order", "申请号"));  
        nvps.add(new BasicNameValuePair("displayCols", "appNumber,appDate")); //支持多个字段用逗号连接；为空返回所有字段
        nvps.add(new BasicNameValuePair("dbs", "FMZL"));
        nvps.add(new BasicNameValuePair("exp", "申请号=('CN201210105520.X')"));
        nvps.add(new BasicNameValuePair("from", "0")); //支持多个字段用逗号连接；为空返回所有字段
        nvps.add(new BasicNameValuePair("to", "1"));
        //服务密码
        //nvps.add(new BasicNameValuePair("serverPassword", NewPatentInterfaceConnection.SERVERPASSWORD));  
        nvps.add(new BasicNameValuePair("machinecode", PatentInterfaceConnection.MACHINECODE));
      //服务器加密时间戳
        Map<String,String> passParam=MD5Util.getEncodeParam();
        if(passParam.size()>0){
            for(String key:passParam.keySet()){
                nvps.add(new BasicNameValuePair(key, passParam.get(key)));
            }
        }
        
        // 设置表单提交编码为UTF-8  
        UrlEncodedFormEntity entry;
        try {
            entry = new UrlEncodedFormEntity(nvps, PatentInterfaceConnection._encode);
            entry.setContentType(PatentInterfaceConnection._application);  
            httppost.setEntity(entry);
        } catch (UnsupportedEncodingException e1) {
            e1.printStackTrace();
        }  
        HttpResponse response;
        try {
            response = httpclient.execute(httppost);
            if(response.getStatusLine().getStatusCode()==200){
                JSONObject resultJson=null;
                HttpEntity entity = response.getEntity();
                if(entity!=null){
                    resultJson=JSONObject.fromObject(EntityUtils.toString(entity, "UTF-8"));
                    EntityUtils.consume(entity);
                }
                if(resultJson!=null&&resultJson.containsKey("message")&&resultJson.getString("message").equals(PatentInterfaceConnection._status)){
                    flag=true;
                    resultJson.clear();
                }
            }
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        finally{
            if(!flag){
                sendMsg();
            }
            nvps.clear();
            httpclient.getConnectionManager().shutdown();
        }
        return flag;
    }


    public class MD5Util {
    
    private final static String keyParamName="keyStr";
    private final static String passParamName="valueStr";
    private final static String key="com.xxyy.zhuanlitong";
    
    /**
     * 获取加密参数字符串
     * @return
     */
    public static String getEncodeParamStr(boolean preSplit){
        StringBuilder sb=new StringBuilder();
        Map<String,String> encodeParam=getEncodeParam();
        if(encodeParam.size()>0){
            for(String key:encodeParam.keySet()){
                sb.append("&").append(key).append("=").append(encodeParam.get(key));
            }
            if(preSplit){
                return sb.substring(1);
            }
        }
        return sb.toString();
    }
    /**
     * 获取加密参数集合对象
     * @return
     */
    public static Map<String,String> getEncodeParam(){
        Map<String,String> param=new HashMap<String,String>();
        long t=System.currentTimeMillis();
        param.put(MD5Util.keyParamName, t+"");
        param.put(MD5Util.passParamName, md5Encode(t+MD5Util.key));
        return param;
    }

    /**
     * <p>MD5加密 生成32位md5码</p>
     * 
     * @param 待加密字符串
     * @return 返回32位md5码
     */
    private static String md5Encode(String inStr) {
        MessageDigest md5 = null;
        byte[] byteArray = null;
        try {
            md5 = MessageDigest.getInstance("MD5");
            byteArray = inStr.getBytes("UTF-8");
        } catch (Exception e) {
            System.out.println(e.toString());
            e.printStackTrace();
            return "";
        }

        byte[] md5Bytes = md5.digest(byteArray);
        StringBuffer hexValue = new StringBuffer();
        for (int i = 0; i < md5Bytes.length; i++) {
            int val = ((int) md5Bytes[i]) & 0xff;
            if (val < 16) {
                hexValue.append("0");
            }
            hexValue.append(Integer.toHexString(val));
        }
        return hexValue.toString();
    }
}
```