---
title: git常见错误
tags: [git]
---

1）RPC failed

提交远程时执行git push命令错误

```
remote: fatal: early EOF
efrror: RPC failed; curl 55 SSL_write() returned SYSCALL, errno = 10053
atal: The remote end hung up unexpectedly
fatal: The remote end hung up unexpectedly
Everything up-to-date
```

解决方法：

```
git config --global http.postBuffer 15728640
```

注：可能的原因是网络问题，或者上传的文件太大，个人感觉是网络问题，多试几次，或者网络良好的情况下尝试提交，应该就不会有这个问题了。