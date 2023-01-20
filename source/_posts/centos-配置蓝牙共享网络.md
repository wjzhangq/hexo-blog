---
title: centos 配置蓝牙共享网络
date: 2015-12-23 12:18:40
tags:
---

安装centos蓝牙库：
```
yum install bluez.x86_64 bluez-compat.x86_64
```
安装蓝牙认证库:
```
yum -y install pygobject2 dbus-python-devel.x86_64 dbus-python
```
配置dhcpd：
```
vim /etc/dhcp/dhcpd.conf

ignore client-updates;

subnet 10.0.0.0 netmask 255.255.255.0 {
    option routers      10.0.0.1;
    option subnet-mask  255.255.255.0;
    option domain-name-servers 10.16.0.222;

    range dynamic-bootp 10.0.0.10 10.0.0.50;
}
```
上网设置：
```
brctl addbr pan1
brctl stp pan1 off
echo "1">/proc/sys/net/ipv4/ip_forward
ifconfig pan1 10.0.0.1/24
iptables -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE
#iptables -D FORWARD 1
#watch -n 1 iptables  -L -n -v
/etc/init.d/dhcpd restart

hciconfig hci0 up piscan
modprobe bnep
pand --listen --role NAP --master
nohup python /root/src/simple-agent.py > /tmp/blue-agent.log 2>&1
```
