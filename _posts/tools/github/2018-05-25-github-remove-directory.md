---
title: 删除github提交的文件夹
tags: [github]
---

场景：提交代码时没有编辑ignore文件，造成target信息也传递到了github上。

1）运行git命令

```
#--cached不会把本地的target删除
git rm -r --cached target
git commit -m "delete target dir"
git push -u origin master
```