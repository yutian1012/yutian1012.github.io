---
title: 持续集成jenkins命令
tags: [jenkins]
---

Jenkins有一个内置的命令行界面，允许用户和管理员从脚本或shell环境访问Jenkins。这对于日常任务的脚本编写，批量更新，故障排除等都是很方​​便的。命令行界面可以通过SSH或Jenkins CLI客户端进行访问，这是Jenkins .jar发布的一个文件。

1）下载jenkins-cli.jar文件

在${home}/.jenkins\war\WEB-INF下存在jenkins-cli.jar，通过该文件能够操作jenkins。

注：可在jenkins服务器中直接下载http://localhost:8080/jnlpJars/jenkins-cli.jar

2）运行命令

```
# 进入jenkins-cli.jar所在的目录（下载后放置的目录）
cd D:\work\installer\jenkins

# -s命令指定jenkins的服务地址，help表示查看帮助命令
java -jar jenkins-cli.jar -s http://localhost:8080/ help

# 运行命令时添加认证参数
java -jar jenkins-cli.jar [-s JENKINS_URL] -auth kohsuke:abc1234ffe4a help
```

3）命令行认证错误提示

```
ERROR: You must authenticate to access this Jenkins
```

服务器认证操作

```
# 使用-auth参数设置认证，konsuke表示用户名，abc1234ffe4a表示密码
java -jar jenkins-cli.jar [-s JENKINS_URL] -auth kohsuke:abc1234ffe4a command 
```

4）常用命令

重启服务

```
java -jar jenkins-cli.jar [-s JENKINS_URL] -auth kohsuke:abc1234ffe4a restart
```

关闭服务

```
java -jar jenkins-cli.jar [-s JENKINS_URL] -auth kohsuke:abc1234ffe4a shutdown
```