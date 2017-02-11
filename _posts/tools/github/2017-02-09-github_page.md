---
title: github_page
tags: [github,tools]
---

git，github和github page

### 1. 什么是Git，GitHub和GitHub Pages
Git is what you do locally on your own computer and GitHub and GitHub Pages  is the place to store the work you finish. 

注：Git是客户端本地使用的工具（版本控制工具），而GitHub和GitHub Page是存放内容的服务器。

1）Git is a version control system that tracks changes to files in a project over time. 

2）GitHub is a web hosting service for the source code of software and web development projects (or other text based projects) that use Git. 放置项目源代码的公共服务器。

3）GitHub Pages are public webpages hosted for free through GitHub.以GitHub存储的project，做个人网站使用。

Pages lets you do the same things as GitHub, but if the repository is named a certain way and files inside it are HTML or Markdown, you can view the file like any other website.

注：githubpage只是存储的项目中包含的文件内容是html或markdown文本。能被内部的jekyll解析成html，进而可以在网页中浏览。

### 2. GitHub Page的使用
注：一下操作都是在githhub官网上操作的

1）登录GitHub，官网：https://github.com/

2）New repository创建一个仓库

![](/images/tools/github/newrepository.png)

3）创建index.html文件，并提交

4）访问该网站：https://yutian1012.github.io/，就可以看到index.html网页内容了。

5）创建样式文件：css/main.css，并提交。

create a new file named css/main.css. The css/ before the filename will automatically create a subdirectory called css。

6）在index.html中关联样式文件，并提交修改

在index.html的<head>标签中添加样式：
```
<link rel="stylesheet" type="text/css" href="/css/main.css">
```
7）再次访问https://yutian1012.github.io/，可以看到添加样式后的网页了

### 3. jekyll+github部署网站
{% highlight haskell %}
以下代码在使用时需要将{ {之间和{ %之间的空格去掉，这里为了防止被jekyll解析特意加的空格。
{% endhighlight %}

1）什么是Jekyll

Jekyll is a very powerful static site generator.能够快速构建静态网站。depends on templates（使用模板生成html文件）。
传统的website在删除，添加页面后，都要修改相应的navigator，而使用Jekyll可以通过layout文件模板来实现。

*注：要使用Jekyll需要在github的repo中创建jekyll的目录结构。*

2）创建.gitignore文件

Create a .gitignore file. This file tells Git to ignore the _site directory that Jekyll automatically generates each time you commit. 

文档内容：
```
_site/
```
注：忽略_site目录，该目录下的文档是上传文件后，jekyll自动生成的html页面。

3）创建_config.yml

Create a _config.yml file that tells Jekyll some basics about your project.

文档内容：
```
name: yutian1012 blog
markdown: kramdown
```

4）创建目录_layouts，并在该目录下创建default.html

Make a _layouts directory, and create file inside it called default.html

default.html是主要的布局文件，应包含所有页面中重复利用的代码，如<head>，<footer>等元素，应把这些元素从index.html中移除。

文档内容：
```
<!DOCTYPE html>
    <html>
        <head>
            <title>{ { page.title } }</title>
            <!-- link to main stylesheet -->
            <link rel="stylesheet" type="text/css" href="/css/main.css">
        </head>
        <body>
            <nav>
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/about">About</a></li>
                    <li><a href="/cv">CV</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            </nav>
            <div class="container">
            
            { { content } }
            
            </div><!-- /.container -->
            <footer>
                <ul>
                    <li><a href="mailto:hankquinlanhub@gmail.com">email</a></li>
                    <li><a href="https://github.com/hankquinlan">github.com/hankquinlan</a></li>
                </ul>
            </footer>
        </body>
    </html>
```

liquid tags标签

```
{ { page.title } }and { { content } }
```

liquid tags，会在生成页面时被填充内容。

5）修改index.html页面，只保留文档内容，并在文档头添加YAML

文档内容：
```
---
layout: default
title: Hank Quinlan, Horrible Cop
---
<div class="blurb">
    <h1>Hi there, I'm Hank Quinlan!</h1>
    <p>I'm best known as the horrible cop from <em>A Touch of Evil</em> Don't trust me. <a href="/about">Read more about my life...</a></p>
</div><!-- /.blurb -->
```

注：layout: default，指定布局文件，Jekyll在生成html时会用该布局模板，并将内容填充到模板中的content占位符。

6）构建blog

creating a post, making a page to list our posts, creating a custom permalink for posts, and creating an RSS feed for the blog.

7）在_layouts目录下创建posts.html，用来在显示博客文章样式的模板文件。

文档内容：
```
---
layout: default
---
<h1>{ { page.title } }</h1>
<p class="meta">{ { page.date | date_to_string } }</p>

<div class="post">
  { { content } }
</div>
```

注：Notice the post layout uses the default layout as it's base。

8）构建_posts目录来存放post(具体博文)。

Make a _posts/ directory where we'll store our blog posts.

创建一个具体的文章：2014-04-30-yutian1012-site-launched.md

文档内容：
```
---
layout: post
title: "Hank Quinlan, Horrible Cop, Launches Site"
date: 2014-04-30
---

Well. Finally got around to putting this old website together. Neat thing about it - powered by [Jekyll](http://jekyllrb.com) and I can use Markdown to author my posts. It actually is a lot easier than I thought it was going to be.
```

访问该文件：http://yutian1012.github.io/2014/04/30/yutian1012-site-launched

注：layout为post，即是用post.html做布局模板。date，title值都会传递到post.html文件的page.title和page.date标签中。

```
post.html文件中的标签{ { page.title } }和{ { page.date | date_to_string } }
```

注2：文档名称格式固定year-month-day-xxx.md

注3：访问地址也固定http://username.github.io/YYYY/MM/DD/name-of-your-post

9）创建博客显示列表，创建blog目录，并在该目录下创建index.html

Create a blog directory and create a file named index.html inside it. To list each post, we'll use a foreach loop to create an unordered list of our blog posts。

文档内容：
```
---
layout: default
title: yutian1012 Blog
---
    <h1>{ { page.title } }</h1>
    <ul class="posts">

      { % for post in site.posts % }
        <li><span>{ { post.date | date_to_string } }</span> » <a href="{ { post.url } }" title="{ { post.title } }">{ { post.title } }</a></li>
      { % endfor % }
    </ul>
```

注：site.posts遍历_posts文件下的md文档。

注2：通过_layouts/default.html文件中定义的连接能定位到blog目录下，因此可以正常显示出我们的博文列表。

```
<li><a href="/blog">Blog</a></li>
```

10）改变文档连接地址

当前文档访问的地址为：http://username.github.io/YYYY/MM/DD/name-of-your-post，在域名后直接是/YYYY。

现在想在域名后添加一个blog，使其连接地址变为：http://username.github.io/blog/YYYY/MM/DD/name-of-your-post

修改_config.yml文档，添加代码
```
permalink: /blog/:year/:month/:day/:title
```

11）在blog目录下创建文件atom.xml

文档内容：
```
---
layout: feed
---
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">

    <title>Hank Quinlan's Blog</title>
    <link href="http://hankquinlan.github.io/blog/atom.xml" rel="self"/>
    <link href="http://hankquinlan.github.io/blog"/>
    <updated>{ { site.time | date_to_xmlschema } }</updated>
    <id>http://hankquinlan.github.io/blog</id>
    <author>
        <name>Hank Quinlan</name>
        <email>hankquinlanhub@gmail.com</email>
    </author>

    { % for post in site.posts % }
        <entry>
            <title>{ { post.title } }</title>
            <link href="http://hankquinlan.github.io{ { post.url } }"/>
            <updated>{ { post.date | date_to_xmlschema } }</updated>
            <id>http://hankquinlan.github.io{ { post.id } }</id>
            <content type="html">{ { post.content | xml_escape } }</content>
        </entry>
    { % endfor % }

</feed>
```

查看：https://yutian1012.github.io/blog/atom.xml

注：该文档是提供给RSS使用的。

### 4. 补充RSS和feed
1）RSS是一种用于共享新闻和其他Web内容的数据交换规范。是站点用来和其他站点之间共享内容的一种简易方式（也叫聚合内容）。

2）Feed：消息来源是一种资料格式，网站透过它将最新资讯传播给用户。用户能够订阅网站的先决条件是，网站提供了消息来源。

3）一个RSS文件就是一段规范的XML数据，该文件一般以rss，xml或者rdf作为后缀。发布一个RSS文件（一般称为RSS Feed）后，这个RSS Feed中包含的信息就能直接被其他站点调用，而且由于这些数据都是标准的XML格式，所以也能在其他的终端和服务中使用，如PDA、手机、邮件列表等。网站提供RSS输出，有利于让用户获取网站内容的最新更新。

4）网络用户可以在客户端借助于支持RSS的聚合工具软件，在不打开网站内容页面的情况下阅读支持RSS输出的网站内容。

5）使用软件FeedDemon订阅Rss

安装FeedDemon软件：http://www.feeddemon.com/
在网络上找到RSS资源复制到订阅栏中，新建订阅。

6）jekyll支持RSS

_config.yml配置文件，gems:- jekyll-feed


参考资料：http://jmcglone.com/guides/github-pages/

参考资料2：https://github.com/jekyll/jekyll-feed
