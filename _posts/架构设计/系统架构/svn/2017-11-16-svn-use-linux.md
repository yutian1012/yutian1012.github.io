---
title: visualSVN server服务安装与使用-linux
tags: [architecture]
---

1）系统环境

采用centos系统作为svn服务器的系统环境，查看系统环境

```
cat /etc/redhat-release
uname -r
```

2）安装svn（subversion）

首先检查svn软件是否已安装

```
rpm -aq subversion

yum -y install subversion
```

注：保留yum下载的软件包

```
sed -i 's#keepcache=0#keepcache=1#g' /etc/yum.conf
grep keepcache /etc/yum.conf
```

即修改yum的配置文件yum.conf，保留yum安装的软件。

3）配置并启动svn

创建svn版本库数据存储根目录（svndata）即用户,密码权限目录（snvpasswd）

```
mkdir -p /application/svndata
mkdir -p /application/svnpasswd
```

启动svn服务指定服务的svn根目录

```
svnserve -d -r /application/svndata

svnserve --help 查看帮助文档

ps -ef | grep svn 查看是否启动成功

netstat -lntup | grep 3690 查看端口号
```

注：-d（daemon）指定进程模式（守护进程），-r（root）指定存储根目录

4）创建项目版本库

创建一个新的subversion项目sadoc，使用svnadmin命令创建，会生成一些初始化文件

```
svnadmin create /application/svndata/sadoc

tree /application/svndata/sadoc 查看创建项目的目录信息
```

5）调整svn配置文件及权限文件

sadoc项目的配置文件位于/application/svndata/sadoc/conf目录下，存在authz，passwd和svnserve.conf文件。svnserve.conf文件主配置文件，passwd是用户和密码文件，authz是权限的管理文件。

```
ll /application/svndata/sadoc/conf 查看目录下的内容

cd /application/svndata/sadoc/conf 切换目录

cp svnserve.conf svnserve.conf.bak 先备份文件，再修改

vi svnserve.conf 编辑文件

anon-access = none 设置匿名访问为null，不允许匿名访问
auth-access=write 访问权限为write
#默认情况下，一个项目都有各自的权限和用户密码管理文件，为了访问可统一管理
password-db=/application/svnpasswd/passwd 将所有的版本库密码文件统一管理
authz-db = /application/svnpasswd/authz 将权限数据库也统一管理

diff svnserve.conf.bak svnserve.conf 比较文件修改情况
```

在/application/svnpasswd中并不存在passwd和authz目录，需要将sadoc项目下的passwd和authz拷贝过去，作为目标。

```
cp passwd authz /application/svnpasswd/

cd /application/svnpasswd
chmod 700 * 设置权限，除了root用户，其他用户都不能访问
```

6）修改用户（passwd）和权限文件（authz）

修改用户和密码文件，

```
vi /application/svnpasswd/passwd

# 在[user]模块下修改
user11 = usersecret 添加用户和密码
```

修改权限文件authz

```
vi /application/svnpasswd/authz

# 管理的权限可精确到目录级别
[<版本库>:/项目/目录]
@<用户组名> = <权限>
<用户名> = <权限>

#权限包括：w,r,wr

#设置sadoc项目的根目录权限，并设置user123对根目录有些权限
[sadoc:/]
user123 = rw

# 设置组，并对组进行授权，在[groups]下创建组，并指定组的成员
[groups]
sagroup = use123,user233
#对组进行授权
[sadoc:/]
@sagroup = rw
```

注：修改passwd和authz不需要重新启动svn服务。

7）svn重新启动

```
pkill svnserve

svnserve -d -r /application/svndata
```

8）客户端tortoiseSVN

客户端分为32位和64位，安装时需要注意。由于安装时采用的是独立进程的方式安装的svn，因此访问时需要使用svn:的方式来连接访问snv服务

```
svn://10.2.3.9/sadoc
```

下载sadoc项目，首先在相应的目录下创建sadoc目录，在改目录上右击，然后checkout。

注：需要先创建项目名称目录

tortoiseSVN在subversion的缺省目录下有三个目录来保存认证信息（%APPDATA%\Subversion\auth）。包括svn.simple，svn.ssl.server，svn.username。

svn.simple包含基本认证方式所需要的认证信息（用户名/密码），保存的密码是通过WinCrypt API加密的，不是文本形式保存的。

svn.ssl.server里包含了SSL服务器证书

svn.username 包含了用户名认证的认证信息（不需要提供密码）

9）linux下svn的客户端管理

```
用法：svn <subcommand> [options] [args]

# 常用命令
svn checkout(co) 从源码库中取出一个工作版本的拷贝
svn commit(ci) 提交
svn update(up) 更新

svn help 查看其它名
```

实际操作

```
mkdir /svndata 在根目录下创建一个svndata目录，作为svn客户端工作目录

svn co svn://10.2.3.6/sadoc /svndata/ --username=user123 --password=usersecret
```

如果存在一个中文的文件，那么在linux下载checkout时会出现问题

```
svn:can't convert string from 'UTF-8' to native encoding

# 设置linux的字符集
LANG=en_US.UTF-8

#查看当前编码命令
locale
```

查看远程svn仓库的数据信息

```
svn ls svn://10.2.3.6/sadoc --username=user123 --password=usersecret
```

提交文件，需要先add，然后在commit

```
touch a.txt //创建文件

svn add a.txt 

svn ci -m 'add file'
```

注：add表示把文件或目录放到版本控制下，当commit后，会添加到版本库中。

创建分支，使用copy命令

```
mkdir -p /svn/trunk /svn/branch /svn/tag

svn import /svn file:///application/svn/sadoc -m 'import directory'

svn copy svn://10.2.3.5/sadoc/trunk svn://10.2.3.5/sadoc/branch/branch_cms_110329 -m 'create a branch'
```

注：import提交一个没有被版本库管理的文件或文件夹，可以不是add+commit，直接上传到svn库中。import第一个参数指定导入的目录或文件（如目录/svn）。