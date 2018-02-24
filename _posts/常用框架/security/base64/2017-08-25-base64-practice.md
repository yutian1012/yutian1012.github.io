---
title: JAVA模拟微博登录以及发微博
tags: [security]
---

参考：https://www.iswin.org/2015/01/15/JAVA-simulation-microblogging-login-and-microblogging/

JAVA模拟微博登录以及发微博，使用微博的用户名和密码登录系统，然后获取登录后的cookie。自动发送微博时需要将cookie信息一并传递过去。

代码实现：

```
import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLEncoder;

import org.apache.commons.codec.binary.Base64;
/***
 * @blog   http://www.iswin.org
 * @author iswin
 */
public class Test {
    public static void main(String[] args) throws Exception {
        // 设置微博用户名以及密码（填写自己的微博用户名和密码）
        String ticket = requestAccessTicket("username", "password");
        if (ticket != "false") {
            System.err.println("获取成功:" + ticket);
            System.err.println("开始获取cookies");
            String cookies = sendGetRequest(ticket, null);
            System.err.println("cookies获取成功:" + cookies);
            System.err.println("开始发送微博");
            sendWeiBoMessage("java robot", cookies);
            System.err.println("发送微博成功");
        } else
            System.err.println("ticket获取失败，请检查用户名或者密码是否正确!");
    }
    /**
     * 获取登录后返回的cookie信息
     * @param url
     * @param cookies
     * @return
     * @throws MalformedURLException
     * @throws IOException
     */
    public static String sendGetRequest(String url, String cookies)
            throws MalformedURLException, IOException {
        HttpURLConnection conn = (HttpURLConnection) new URL(url).openConnection();
        conn.setRequestProperty("Cookie", cookies);
        conn.setRequestProperty("Referer","http://login.sina.com.cn/signup/signin.php?entry=sso");
        conn.setRequestProperty(
                "User-Agent",
                "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:34.0) Gecko/20100101 Firefox/34.0");
        conn.setRequestProperty("Content-Type",
                "application/x-www-form-urlencoded");
        BufferedReader read = new BufferedReader(new InputStreamReader(
                conn.getInputStream(), "gbk"));
        String line = null;
        StringBuilder ret = new StringBuilder();
        while ((line = read.readLine()) != null) {
            ret.append(line).append("\n");
        }
        StringBuilder ck = new StringBuilder();
        try {
            for (String s : conn.getHeaderFields().get("Set-Cookie")) {//获取cookie
                ck.append(s.split(";")[0]).append(";");
            }
        } catch (Exception e) {
        }
        return ck.toString();
    }
    /**
     * 返回登录的连接地址（重定向地址），其中包含ticket信息
     * @param username
     * @param password
     * @return
     * @throws MalformedURLException
     * @throws IOException
     */
    public static String requestAccessTicket(String username, String password)
            throws MalformedURLException, IOException {
        username =new String(Base64.encodeBase64(username.replace("@", "%40").getBytes()));
        System.out.println(username);
        HttpURLConnection conn = (HttpURLConnection) new URL(
                "https://login.sina.com.cn/sso/login.php?client=ssologin.js(v1.4.15)").openConnection();
        conn.setDoInput(true);
        conn.setDoOutput(true);
        conn.setRequestMethod("POST");
        conn.setRequestProperty("Referer",
                "http://login.sina.com.cn/signup/signin.php?entry=sso");
        conn.setRequestProperty(
                "User-Agent",
                "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:34.0) Gecko/20100101 Firefox/34.0");
        conn.setRequestProperty("Content-Type",
                "application/x-www-form-urlencoded");
        DataOutputStream out = new DataOutputStream(conn.getOutputStream());
        
        //format后面的参数会替换掉前面的%s（占位符）
        out.writeBytes(String.format("entry=sso&gateway=1&from=null&savestate=30&useticket=0&pagerefer=&vsnf=1&su=%s&service=sso&sp=%s&sr=1280*800&encoding=UTF-8&cdult=3&domain=sina.com.cn&prelt=0&returntype=TEXT",
                        URLEncoder.encode(username), password));
        out.flush();
        out.close();
        BufferedReader read = new BufferedReader(new InputStreamReader(conn.getInputStream(), "gbk"));
        String line = null;
        StringBuilder ret = new StringBuilder();
        while ((line = read.readLine()) != null) {
            ret.append(line).append("\n");
        }
        String res = null;
        try {
            res = ret.substring(ret.indexOf("https:"),ret.indexOf(",\"https:") - 1).replace("\\", "");
        } catch (Exception e) {
            e.printStackTrace();
            res = "false";
        }
        return res;
    }
    /**
     * 发送微博消息
     * @param message
     * @param cookies
     * @return
     * @throws MalformedURLException
     * @throws IOException
     */
    public static String sendWeiBoMessage(String message, String cookies)
            throws MalformedURLException, IOException {
        HttpURLConnection conn = (HttpURLConnection) new URL(
                "http://www.weibo.com/aj/mblog/add?ajwvr=6").openConnection();
        conn.setDoInput(true);
        conn.setDoOutput(true);
        conn.setRequestMethod("POST");
        conn.setRequestProperty("Cookie", cookies);
        conn.setRequestProperty("Referer",
                "http://www.weibo.com/u/2955825224/home?topnav=1&wvr=6");
        conn.setRequestProperty("X-Requested-With", "XMLHttpRequest");
        conn.setRequestProperty(
                "User-Agent",
                "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:34.0) Gecko/20100101 Firefox/34.0");
        conn.setRequestProperty("Content-Type",
                "application/x-www-form-urlencoded");
        DataOutputStream out = new DataOutputStream(conn.getOutputStream());
        out.writeBytes("location=v6_content_home&appkey=&style_type=1&pic_id=&text="
                + URLEncoder.encode(message)
                + "&pdetail=&rank=0&rankid=&module=stissue&pub_type=dialog&_t=0");
        out.flush();
        out.close();
        BufferedReader read = new BufferedReader(new InputStreamReader(
                conn.getInputStream(), "gbk"));
        String line = null;
        StringBuilder ret = new StringBuilder();
        while ((line = read.readLine()) != null) {
            ret.append(line).append("\n");
        }
        return ret.toString();
    }
}
```