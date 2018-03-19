---
title: 虚拟机网络连接
tags: [linux]
---

参考：http://blog.csdn.net/u012110719/article/details/42318717

参考：http://blog.sina.com.cn/s/blog_716844910100r3o7.html

网络连接中的虚拟网络：

在安装完成虚拟机后，会在网络连接中出现很多网络连接。其中包括VMnet1，VMnet8等。

![](/images/linux/vm/vm-net.png)

NAT模式下的是VMnet8虚拟网络，host-only模式下的是VMnet1虚拟网络，以及bridged模式下的是VMnet0虚拟网络。这些虚拟网络都是由VMWare虚拟机自动配置而生成的，不需要用户自行设置。VMnet8和VMnet1提供DHCP服务，VMnet0虚拟网络则不提供。

1）如何配置

菜单--编辑--虚拟网络编辑器

![](/images/linux/vm/vm-net-edit.png)

![](/images/linux/vm/vm-net-edit-vmnet8.png)

2）VMnet0

vmnet0，实际上就是一个虚拟的网桥

![](/images/linux/vm/vm-net-bridge.png)

vmnet0，实际上就是一个虚拟的网桥，这个网桥有很若干个端口，一个端口用于连接你的Host，一个端口用于连接你的虚拟机，他们的位置是对等的，谁也不是谁的网关。所以在Bridged模式下，你可以让虚拟机成为一台和你的Host相同地位的机器。

3）VMnet1

vmnet1，是一个Host-Only网络模式

![](/images/linux/vm/vm-net-host.png)

vmnet1，用于建立一个与世隔绝的网络环境所用到的，其中vmnet1也是一个虚拟的交换机，交换机的一个端口连接到你的Host上，另外一个端口连接到虚拟的DHCP服务器上（实际上是vmware的一个组件），另外剩下的端口就是连虚拟机了。

虚拟网卡 “VMWare Virtual Ethernet Adapter for VMnet1”作为虚拟机的网关接口，为虚拟机提供服务。在虚拟机启动之后，如果你用ipconfig命令，你会很清楚的看到，你的默认网关就是指向 “VMWare Virtual Ethernet Adapter for VMnet1”网卡的地址的。（实际上它并不能提供路由，这是VMware设计使然），这里没有提供路由主要表现在没有提供NAT服务，使得虚拟机不可以访问Host-Only模式所指定的网段之外的地址。

4）VMnet8

vmnet8，这是一个NAT方式，最简单的组网方式。

![](/images/linux/vm/vm-net-nat.png)

vmnet8，从主机的“VMWare Virtual Ethernet Adapter for VMnet8”虚拟网卡出来，连接到vmnet8虚拟交换机，虚拟交换机的另外的口连接到虚拟的NAT服务器（这也是一个Vmware组件），还有一个口连接到虚拟DHCP服务器，其他的口连虚拟机，虚拟机的网关即是“VMWare Virtual Ethernet Adapter for VMnet8”网卡所在的机器，即Host宿主机器。同样，用ipconfig也可以看出来，你的虚拟机的默认网关也指向了你的 “VMWare Virtual Ethernet Adapter for VMnet8”虚拟网卡地址。相比之下，可以看出来，NAT组网方式和Host-Only方式，区别就在于是否多了一个NAT服务。