---
title: 阿里云gre模式组网
date: 2015-11-25 12:16:45
tags:
    - LINUX
---

问题：公司租用多台阿里云主机，只有1台对外服务，其他服务器只能访问内网。内网机器偶尔需要访问外网，但又不希望开通外网访问权限。  
解决方案：将阿里云服务器重新组建一个虚拟局域网，有外网权限的服务器做为网关，其他内网服务器通过网关就能访问外部网络。  

穿透阿里的内网可选用ipip tunnel 或者gre模式，本文选用通用性比较好的gre模式。  

外网机器：  
a. 打开ipv4转发：
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```
b. 增加转发规则：
```
iptables -t nat -A POSTROUTING -s 192.168.122.0/24 ! -d 192.168.122.0/24 -j MASQUERADE
```
c. 建立内网隧道：
```
modprobe ip_gre  #加载gre内核模块
ip tunnel add gre3 mode gre remote 10.10.10.3 local 10.10.10.2  ttl 64  #建立隧道
ip link set gre3 up #启动虚拟网卡
ip addr add 192.168.122.2 peer 192.168.122.3 dev gre3 #设定路由
```
内网需要上网机器：  
a. 建立隧道
```
modprobe ip_gre  #加载gre内核模块
ip tunnel add gre1 mode gre remote 10.10.10.2 local 10.10.10.3  ttl 64  #建立隧道
ip link set gre1 up #启动虚拟网卡
ip addr add 192.168.122.3 peer 192.168.122.2 dev gre1 #设定路由
```
b. 设定默认路由
```
route del default
route add default gw 192.168.122.2
```
ps停用隧道方式：
```
ip link set gre1 down
ip tunnel del gre1
```