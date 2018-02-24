---
title: RSS-java解析
tags: [web]
---

以百度RSS为例，使用java解析获取RSS发布的信息。

```
http://news.baidu.com/n?cmd=4&class=civilnews&tn=rss
```

注：使用迅雷下载上面的网址，打开可以发现是一个xml文档。

1）创建maven项目，并添加依赖

使用rome解析，需要添加依赖的jar包：jdom-1.0.jar和rome-1.0.jar。其中rome-1.0.jar内部依赖jdom。

```
<dependency>
    <groupId>com.rometools</groupId>
    <artifactId>rome</artifactId>
    <version>1.9.0</version>
</dependency>
```

注：jdom依赖包会自动下载。

2）实体类代码：

包括title标题，author作者，uri资源标识，link连接，description描述信息，pubDate发布时间，type信息类型（如新闻，娱乐等）。

```
/**
 * rss中的条目对象--一个条目就代表一篇文章
 */
public class RssItemModel {
    private String title;  
    private String author;  
    private String uri;  
    private String link;  
    private String description;  
    private Date pubDate;  
    private String type;
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public String getAuthor() {
        return author;
    }
    public void setAuthor(String author) {
        this.author = author;
    }
    public String getUri() {
        return uri;
    }
    public void setUri(String uri) {
        this.uri = uri;
    }
    public String getLink() {
        return link;
    }
    public void setLink(String link) {
        this.link = link;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    public Date getPubDate() {
        return pubDate;
    }
    public void setPubDate(Date pubDate) {
        this.pubDate = pubDate;
    }
    public String getType() {
        return type;
    }
    public void setType(String type) {
        this.type = type;
    }
    @Override
    public String toString() {
        return "RssItemModel [title=" + title + ", author=" + author + ", uri="
                + uri + ", link=" + link + ", description=" + description
                + ", pubDate=" + pubDate + ", type=" + type + "]";
    }  
}
```

3）解析类：

XmlReader对象接收一个RssUrl地址，用来获取外网的xml文档。SyndFeedInput通过XmlReader构建SyndFeed对象，最后通过该对象获取xml中的实体集合。

```
import java.io.Reader;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;

import com.rometools.rome.feed.synd.SyndEntry;
import com.rometools.rome.feed.synd.SyndFeed;
import com.rometools.rome.io.SyndFeedInput;
import com.rometools.rome.io.XmlReader;

/**
 * Rome解析RSS，内部依赖jdom
 */
public class RssReader {
    public static List<RssItemModel> getAllRssItemBeanList(String rssUrl){  
        try{  
            //设置检索的RssURL地址
            URL feedUrl = new URL(rssUrl);  
            //构建XmlReader对象，并传递url地址
            XmlReader xmlReader=new XmlReader(feedUrl);
            //创建SyndFeedInput对象，该对象可关联XmlReader，从xml中获取实体集合
            SyndFeedInput input = new SyndFeedInput(); 
            SyndFeed feed = input.build((Reader)xmlReader);  
            
            //获取xml文档中的实体集合
            List<SyndEntry> entries = feed.getEntries();  
              
            RssItemModel item = null;  
            List<RssItemModel> rssItemBeans = new ArrayList<RssItemModel>();  
              
            for(SyndEntry entry : entries){  
                item = new RssItemModel();  
                item.setTitle(entry.getTitle().trim());  
                item.setType(feed.getTitleEx().getValue().trim());  
                item.setUri(entry.getUri());    
                item.setPubDate(entry.getPublishedDate());  
                item.setAuthor(entry.getAuthor());  
                rssItemBeans.add(item);
            }  
            return rssItemBeans;  
              
        }catch(Exception e){
            e.printStackTrace();  
            return null;  
        }  
    } 
    
    public static void main(String[] args) {
        String url="http://news.baidu.com/n?cmd=4&class=civilnews&tn=rss";
        List<RssItemModel> list=getAllRssItemBeanList(url);
        if(null!=list){
            for(RssItemModel item:list){
                System.out.println(item);
            }
        }
    }
}
```

