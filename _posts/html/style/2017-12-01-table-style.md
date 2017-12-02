---
title: table设置样式
tags: [html]
---

问题1：table控件的内容被撑大

在table元素上设置样式

```
style="table-layout: fixed;word-break: break-all;"
```

注：在火狐和IE上可以正常显示，IE将word-break自动转成了

```
style="table-layout: fixed; -ms-word-break: break-all;"
```

但是火狐不支持-ms-word-break。

问题2：td，div，span中的内容自动换行防止撑大

```
style="word-wrap:break-word;word-break:break-all;"
```