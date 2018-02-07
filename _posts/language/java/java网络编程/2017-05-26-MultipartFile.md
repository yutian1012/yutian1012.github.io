---
title: SimpleCaptcha验证码
tags: [web]
---

### MultipartFile转成File

参考：http://www.cnblogs.com/hahaxiaoyu/p/5102900.html

上传文件类MultipartFile

```
org.springframework.web.multipart.MultipartFile;

multipartFile.transferTo(file);
```


### 上传多个文件

```
MultipartHttpServletRequest request...
Map<String, MultipartFile> files = request.getFileMap();
Iterator<MultipartFile> it = files.values().iterator();

```