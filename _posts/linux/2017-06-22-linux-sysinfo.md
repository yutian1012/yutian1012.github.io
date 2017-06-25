---
title: linux系统信息
tags: [linux]
---

### 查询系统和版本信息

```
uname -a
```

![](/images/linux/sys/uname.png)

### 查看正在运行的内核版本

```
cat /proc/version
```

![](/images/linux/sys/proc-version.png)

注：会出现

### 查看linux发行版本信息

```
cat /etc/issue
```

![](/images/linux/sys/etc-issue.png)

### 查看linux版本

```
lsb_release -a
```

![](/images/linux/sys/lsb_release.png)

注：如果找不到lsb_release命令，可以使用yum安装再使用

```
yum install lsb -y
```