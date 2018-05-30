---
title: gvim底端命令模式-删除
tags: [vim]
---

1）删除包含特定字符的行：

```
g/pattern/d   
```

2）删除不包含指定字符的行：

```
v/pattern/d

g!/pattern/d
```