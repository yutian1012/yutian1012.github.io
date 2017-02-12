---
title: gem和bundler的关系
tags: [jekyll]
---

gem和bundler的关系

### gem
1）什么是gem
gem是一种文件组织的包，一般的ruby的很多插件都有由这种各种的包提供。

2）gem常用命令

gem -v #gem版本，

gem update #更新所有包，

gem install rake #安装rake,从本地或远程服务器 ，

gem help #提醒式的帮助

gem list #查看安装的组件

注：gem是Ruby环境中的包管理器,好比于Python中的pip与JavaScript本地Node.js的npm

### bundler
1）什么是bundler

使用bundler管理多版本的Gem。Bundler使用Ruby语言写的，通过跟踪和安装运行Ruby项目所需要的确切的gem和版本,为Ruby项目提供了完整的可运行环境。

2）bundler的安装：gem install bundler

3）bundler的使用

bundle show，会显示当前ruby项目需要的运行环境

bundle install,会安装当前ruby项目需要的组件，安装完成后可以通过gem list查看到。

注：bundler与Gemfile密切相关，项目中存在Gemfile文件才可以使用bundle命令，并且Gemfile文件中定义了项目依赖的组件，这些组件将交由bundler来管理和配置运行环境。

### 实例
场景一：

```
mkdir app1; cd app1;
echo "source 'https://ruby.taobao.org/'" > Gemfile
echo "gem 'rails,'4.1.0'" >> Gemfile
bundle install
```

app1下安装了rails 4.1.0。

```
bundle exec rails -v
```

查看当前目录下使用的rails版本，显示内容应该为Rails 4.1.0

```
bundle exec rails new . --force
```

覆盖原来Gemfile,此时的app使用的rails版本为4.1.0

场景二：

```
mkdir app2; cd app2;
echo "source 'https://ruby.taobao.org/'" > Gemfile
echo "gem 'rails,'3.2.13'" >> Gemfile
bundle install
```

创建了第二个app2文件夹，并通过bundler安装了rails 3.2.13 同样通过bundle exec rails new . --force可以生成基于rails 3.2.13版本的应用。

注：安装了以上两个版本后，通过gem list --local可以看到rails有两个版本，显示为rails (4.1.0, 3.2.13),bundler会智能的判断每个项目的rails版本，以确保应用的正确运行。
