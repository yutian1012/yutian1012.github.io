---
title: tomcat常见问题
tags: [tomcat]
---

### Failed to process jar [jar:file…jar!/] for TLD

错误信息

```
Failed to process JAR [jar:file:/D:/Program%20Files/apache-tomcat-7.0.59/webapps/zsy/WEB-INF/lib/joda-time-1.6.2.jar!/] for TLD files
java.util.zip.ZipException: error in opening zip file
```

将maven中出错的jar包在本地仓库中删除后，又重新下载后，再次部署项目就没有问题了。

### Caused by: java.lang.OutOfMemoryError: GC overhead limit exceeded

参考：http://www.cnblogs.com/hucn/p/3572384.html

设置参数：

```
-XX:-UseGCOverheadLimit
```

