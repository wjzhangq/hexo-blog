---
title: 公司vpn配置
date: 2015-07-17 20:49:40
tags:
---

### 服务器：

公网服务器A(ip:121.199.x.x) centos6  
内网测试服务器B(ip:10.10.1.160) centos6  
网络：

公网 121.199.x.x/24  
内网 10.10.1.1/24  
vpn网络 10.10.16.1/24  

需求：

公网服务器A需要能访问内网，因为svn服务器在内网。  
vpn拨号后需要能访问到内网  
方案：

vpn 服务架设在公网服务器A上（有公网ip）,并分配vpn IP：10.10.16.1，  
内网测试服务器B vpn拨号连接到服务器A，分配vpn ip：10.10.16.2，虚拟网卡ppp0  
服务器A增加路由增加内网ip 10.10.1.1/24 路由到 10.10.16.2  
内网测试服务器B增加转发规则将ppp0流量转发到内网  
### 步骤：
1. 开启公网服务器A和内网测试服务器B的ip_forward
```shell
> vim /etc/sysctl.conf
net.ipv4.ip_forward=1
> sysctl -p
```
2. 在公网服务器A安装vpn服务
```shell
> yum install pptpd.x86_64
```
配置：
```shell
> vim /etc/pptpd.conf
localip 10.10.16.1
remoteip 10.10.16.3-20,10.10.16.245
> vim /etc/ppp/options.pptpd
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```
账号设置
```shell
> vim /etc/ppp/chap-secrets
# client server secret IP addresses
vpn pptpd xxx 10.10.16.2
```
重启vpn服务
```shell
> service pptpd restart
```
3. 公网服务器A设置内网路由
```shell
> sudo route add -net 10.10.1.0/24 gw 10.10.16.2 dev ppp0
```
4. 公网服务器设置转发规则
```
> iptables -A POSTROUTING -s 10.10.16.0/24 ! -d 10.10.16.0/24 -j MASQUERADE
 
```
5. 内网测试服务器B 安装vpn客户端
```
> yum install pptpsetup pptp
```
6. 内网测试服务器B拨号
```
> pptpsetup --create test1 --server admin.wefit.com.cn --username vpn --password xxx --encrypt --start
```
7. 内网测试服务器B流量转发
```
> iptables -A POSTROUTING -s 10.10.16.0/24 ! -d 10.10.16.0/24 -j MASQUERADE
```
8. 内网测试服务B FORWARD
```
> iptables -I FORWARD -i ppp0 -o br0 -j ACCEPT
```
 

ps:
>如果客户端提示：  
LCP: timeout sending Config-Requests  
sudo modprobe nf_conntrack_pptp  