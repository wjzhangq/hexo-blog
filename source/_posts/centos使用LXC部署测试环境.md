---
title: centos使用LXC部署测试环境
date: 2015-07-09 18:43:04
tags:
    - LINUX
---

测试服务器为centos7系统。  
使用场景需求：  
每一个开发人员有一个独立虚拟linux系统（系统要求centos6）（方便随便折腾）  
虚拟系统需要独立内网ip

lxc是最优方案，对资源消耗最少，并且每台虚拟机可以共享所有资源。例如我们测试服务器有16G内存，每台虚拟机都能看到16G内存。  
虚拟系统需要独立ip，需要使用桥接模式。

lxc安装：
```shell
> yum install lxc.x86_64 lxc-templates.x86_64
```
guest系统安装：
1. 查看可选的系统：
```shell
> ls /usr/share/lxc/templates
lxc-alpine lxc-archlinux lxc-centos lxc-debian lxc-fedora lxc-openmandriva lxc-oracle lxc-sshd lxc-ubuntu-cloud
lxc-altlinux lxc-busybox lxc-cirros lxc-download lxc-gentoo lxc-opensuse lxc-plamo lxc-ubuntu
```
2. 我需要的centos6， 模版中没有，选择下载系统：
```
> lxc-create -t download -n template
Setting up the GPG keyring
Downloading the image index

—
DIST RELEASE ARCH VARIANT BUILD
—
centos 6 amd64 default 20150619_02:16
centos 6 i386 default 20150619_02:16
centos 7 amd64 default 20150619_02:16
debian jessie amd64 default 20150619_19:16
debian jessie armel default 20150620_22:42
debian jessie armhf default 20150620_22:42
debian jessie i386 default 20150619_19:16
debian sid amd64 default 20150619_19:16
debian sid armel default 20150620_22:42
debian sid armhf default 20150620_22:42
debian sid i386 default 20150619_19:16
debian squeeze amd64 default 20150619_19:16
debian squeeze armel default 20150620_22:42
debian squeeze i386 default 20150619_19:16
debian wheezy amd64 default 20150619_19:16
debian wheezy armel default 20150620_22:42
debian wheezy armhf default 20150620_22:42
debian wheezy i386 default 20150619_19:16
fedora 19 amd64 default 20150618_01:27
fedora 19 armhf default 20150621_01:27
fedora 19 i386 default 20150619_01:27
fedora 20 amd64 default 20150619_01:27
fedora 20 armhf default 20150621_01:27
fedora 20 i386 default 20150619_01:27
gentoo current amd64 default 20150619_14:12
gentoo current armhf default 20150620_14:12
gentoo current i386 default 20150619_14:12
oracle 6.5 amd64 default 20150619_11:40
oracle 6.5 i386 default 20150619_11:40
plamo 5.x amd64 default 20150618_21:36
plamo 5.x i386 default 20150618_21:36
ubuntu precise amd64 default 20150619_19:15
ubuntu precise armel default 20150621_03:49
ubuntu precise armhf default 20150621_03:49
ubuntu precise i386 default 20150619_19:15
ubuntu trusty amd64 default 20150619_19:15
ubuntu trusty arm64 default 20150604_03:49
ubuntu trusty armhf default 20150621_03:49
ubuntu trusty i386 default 20150619_03:49
ubuntu trusty ppc64el default 20150621_03:49
ubuntu utopic amd64 default 20150619_19:15
ubuntu utopic arm64 default 20150605_03:49
ubuntu utopic armhf default 20150621_03:49
ubuntu utopic i386 default 20150619_19:15
ubuntu utopic ppc64el default 20150621_03:49
ubuntu vivid amd64 default 20150619_19:15
ubuntu vivid arm64 default 20150604_03:49
ubuntu vivid armhf default 20150621_03:49
ubuntu vivid i386 default 20150619_19:15
ubuntu vivid ppc64el default 20150621_03:49
ubuntu wily amd64 default 20150619_03:49
ubuntu wily arm64 default 20150604_03:49
ubuntu wily armhf default 20150621_03:49
ubuntu wily i386 default 20150619_19:15
ubuntu wily ppc64el default 20150621_03:49
—

Distribution:
```
3. 等待系统下载完成
```shell
> lxc-ls
template
```

4. 设置桥接模式：
```shell
> vim /etc/sysconfig/network-scripts/ifcfg-enp3s0f0
TYPE=Ethernet
DEVICE=enp3s0f0
ONBOOT=yes # 开机自动启动
BRIDGE=br0 # 这是重点，不用设置ip, 需要新增br0网桥

> vim /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
TYPE=Bridge #网桥
ONBOOT=yes
BOOTPROTO=dhcp ＃使用dhcp获取ip
```
5. 设置虚拟机配置
```shell
> /var/lib/lxc/template/config
lxc.arch = x86_64
lxc.rootfs = /var/lib/lxc/template/rootfs
lxc.utsname = template

# Network configuration
lxc.network.type = veth #桥接
lxc.network.flags = up #开机启动
lxc.network.link=br0 #桥接网桥
lxc.network.hwaddr=00:00:00:00:00:02 #一定要设置，方便dhcp设置

lxc.start.auto = 1
lxc.start.delay = 5
lxc.start.order = 100
```
5. 启动
```
> lxc-start -n template
进入虚拟机，查看虚拟机ip
```
6. 创建更多虚拟机
```
> lxc-clone template test
```

7. 查看每台虚拟机使用情况
```
> lxc-top
```
ps:
>虚拟机根目录：/var/lib/lxc/template/rootfs
改变虚拟机账户密码： chroot /var/lib/lxc/template/rootfs passwd