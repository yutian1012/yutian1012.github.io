---
title: github_page_lemma-theme
tags: [github,tools]
---

github使用lemma-theme模板

### 1. theme模板

主题模板地址：

http://jekyllthemes.org/themes/lemma-theme/

https://github.com/neizod/lemma-theme

### 2. 将theme克隆到本地
```
cd E:\work\blog
git clone https://github.com/neizod/lemma-theme.git
```
命令运行结束后在当前目录下会生成lemma-theme文件夹

### 3. 本地运行jekyll
问题：main.coffee文件没有生成相应的js，修改config.yml文件，在文件最后面添加代码
```
gems: - jekyll-coffeescript
```
问题2：分页无法正常显示，修改config.yml文件，添加 - jekyll-paginat

修改_configy.yml文件的名称和地址等字段，并在末尾追加
```
gems:
 - jekyll-coffeescript
 - jekyll-paginate
```
运行命令jekyll serve启动，访问地址http://localhost:4000

![](/images/tools/github/lemma-themeclone.png)

### 4. 将博客放置到_posts文件夹中
博客文章可以以文件夹的组织形式放入，jekyll会遍历所有的文件夹，生成相应的html文件。html文件会通过标签category分类。

### 5. 将theme上传到你的github仓库中，作为博客站点
1）新建一个空的仓库，仓库名为：yourname.github.io，如果已经存在可先删除再创建。

2）在本地仓库中提交上面的修改
```
git status
git add .
git commit -m 'init blog theme'
```
第一条命令查看当前仓库的状态，会显示代码变动情况。

第二条命令将所有的修改的内容add到仓库中

第三条命令将所有的修改提交到仓库中。

![](/images/tools/github/gitadd.png)

3）修改本地仓库的远程指向
```
git remote -v
git remote set-url origin https://github.com/yourgithub/yourgithub.github.io.git
```

![](/images/tools/github/gitremote.png)

4）查看当前分支并提交（因为我们clone的别人的仓库，分支不一定是master）
```
git push -u origin master
```
*注：git push origin master的意思就是上传本地当前分支代码到master分支*

错误信息:
```
E:\work\blog\lemma-themes>git push origin master
error: src refspec master does not match any.
error: failed to push some refs to 'https://github.com/yutian1012/yutian1012.github.io.
```
原因是本地没有主分支，本地所在的分支查看命令
```
git branch
```

![](/images/tools/github/gitbranch.png)

运行push命令：
```
git push -u origin gh-pages:master
```
将本地仓库的内容提交到github仓库中

![](/images/tools/github/gitpush.png)

### 6. 在本地仓库添加博客并上传到github上
1）将远程的文档下载clone到E:\work\blog目录下
```
git clone https://github.com/yutian1012/yutian1012.github.io.git
```
注：会在当前目录下生成yutian1012.github.io的文件夹，我们的博客文章要放置在该文件夹下的_posts文件夹中。

2）创建文档并命名为2017-02-07github_page_套用theme.md

编辑文档开头，设置分类和标题
```
---
title: github_page_套用theme
tags: [git,tools]
---
```

3）在images文件夹下放置文档所需图

4）上传到github上
```
git add .
git commit -m 'update blog post'
git push origin master
```
![](/images/tools/github/gitaddblog.png)

![](/images/tools/github/gitcommitblog.png)

![](/images/tools/github/gitpushblog.png)

5）上传前最好先在本地运行测试
```
jekyll serve
```

6）图片样式问题，图片大小与页面尺寸不成比例

修改css\main.sass文件，在文件样式开头添加
```
img 
    width: 100%
```