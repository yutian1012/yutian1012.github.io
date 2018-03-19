---
title: linux系统中磁盘的表示
tags: [linux]
---

硬盘的表示方法，硬盘被挂接到/dev下，使用df命令可查看硬盘分区的信息

```
# 查看分区信息
df
-------------------------------------------------------
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/sda2      ext4   8.6G  686M  7.5G   9% /
tmpfs          tmpfs  495M     0  495M   0% /dev/shm
/dev/sda1      ext4   190M   27M  154M  15% /boot
---------------------------------------------------------
```

![](/images/linux/disk/df-hT.png)

1）针对sata,scsi,sas类型的磁盘，以sd开头。

a.硬盘数的表示

sda，sdb，sdc，其中a，b，c表示第几块硬盘。

b.硬盘中分区的表示

sda1，sda2，sda3，其中1，2，3表示第几个分区。

2）针对IDE类型的磁盘，以hd开头

a.硬盘数的表示

hda，hdb，hdc，其中a，b，c表示第几块硬盘。

b.硬盘中分区的表示

hda1，hda2，hda3，其中1，2，3表示第几个分区。