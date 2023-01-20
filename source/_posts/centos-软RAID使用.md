---
title: centos 软RAID使用
date: 2015-08-05 12:59:40
tags:
    - LINUX
---

本次选用raid1方案，其实就是数据互备，最大限度保障数据安全。  
raid 磁盘 /dev/sdb /dev/sdc

1. 安装raid管理软件:
```
> yum install -y parted mdadm
```
2. 将raid磁盘设置为raid格式
```
> fdisk /dev/sdb
>> n # 回车
>> p # 回车
>> 一直回车
>> t # 回车
>> fd # 输入设置raid格式
>> w # 保持并推出
/dev/sdc 也相同操作方式
```

3. 创建raid
在操作之前先来看看这个命令的参数
 - -C #创建软件RAID
 - -l #指定RAID级别
 - -n #指定磁盘个数
 - -x #指定备用设备个数
```
> mdadm -C /dev/md1 -l 1 -n 2 /dev/sdb1 /dev/sdc1
```
4. 开机自动
```
> mdadm -Evs >> /etc/mdadm.conf
```
5. 查看详情
```
> mdadm --detail /dev/md1
```
6. 格式化
```
> mkfs.ext4 /dev/md1
```
7. 挂载
```
> mkdir /data
> mount /dev/md1 /data
```
8. 设置开机自动挂载
```
> echo "/dev/md1 /raid1 ext4 defaults 0 0" >> /etc/fstab
```
