---
title: java前瞻和后瞻
tags: [java]
---

参考：https://www.cnblogs.com/dong-xu/p/6926064.html

在正则表达式当中有个东西叫做前瞻，有的管它叫零宽断言。

1）语法

正确理解向前匹配和向后匹配的方向性。向前指正则继续匹配的方向（即右侧），向后匹配指正则已经匹配过的方向（即左侧）。与我们理解的前后正好相反。

```
(?=X) X, via zero-width positive lookahead 
(?!X) X, via zero-width negative lookahead 
(?<=X) X, via zero-width positive lookbehind 
(?<!X) X, via zero-width negative lookbehind 

表达式         名称          描述
(?=exp)     正向前瞻    匹配后面满足表达式exp的位置
(?!exp)     负向前瞻    匹配后面不满足表达式exp的位置
(?<=exp)    正向后瞻    匹配前面满足表达式exp的位置
(?<!exp)    负向后瞻    匹配前面不满足表达式exp的位置
```

2）实例

![](/images/java_basic/regex/zero-width.png)

```
@Test
public void test(){
    String test="<xml><ToUserName>< ![CDATA[toUser] ]></ToUserName><FromUserName>< ![CDATA[fromUser] ]></FromUserName><CreateTime>1348831860</CreateTime><MsgType><![CDATA[image]]></MsgType><PicUrl><![CDATA[this is a url]]></PicUrl><MediaId><![CDATA[media_id]]></MediaId><MsgId>1234567890123456</MsgId></xml>";
    
    //Pattern p=Pattern.compile("\\s+(?=!\\[CDATA)");//向后查找
    Pattern p=Pattern.compile("(?<=\\])\\s+");
    Matcher m=p.matcher(test);
    if(m.find()){
        System.out.println(m.groupCount());
        System.out.println(m.group(0));
    }
    //直接利用正则替换字符串
    System.out.println(test.replaceAll("\\s+(?=!\\[CDATA)", "")
        .replaceAll("(?<=\\])\\s+", ""));
}

```
