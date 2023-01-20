---
title: Mac Mini 安装centos系统
date: 2015-07-03 15:14:21
tags:
    - LINUX
---

准备把Mac Mini改造为测试服务器。线上环境是Centos6, 所以测试环境也需要安装为Centos 6。Mac OSX 系统需要完全废弃。  
Mac Mini使用EFI引导，只有Centos 7才支持EFI，所以考虑Mac Mini 安装Centos 7，通过lxc虚拟Centos 6

### 安装步骤：  
1. 下载Centos(下载地址http://www.centos.org/download/)：
```shell
wget http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso
```
2. 转化镜像为Mac专用镜像模式(需要Mac 电脑)
```shell
hdiutil convert -format UDRW -o ~/Downloads/CentOS-7-x86_64-Minimal-1503-01.img ~/Downloads/CentOS-7-x86_64-Minimal-1503-01.iso
```

>正在读取CentOS-7- （Apple_ISO：0）…
…………………………………………………………………………………………………………………………………………………………………………………………………………
正在读取（Apple_Free：1）…
经过时间： 1.236s
速度：309.6M 字节/秒
节省：0.0%

3. 插入空白U盘，查看系统u盘设备描述符：
```shell
> diskutil list
/dev/disk0
#: TYPE NAME SIZE IDENTIFIER
0: GUID_partition_scheme *251.0 GB disk0
1: EFI EFI 209.7 MB disk0s1
2: Apple_CoreStorage 250.1 GB disk0s2
3: Apple_Boot Recovery HD 650.0 MB disk0s3
/dev/disk1
#: TYPE NAME SIZE IDENTIFIER
0: Apple_HFS Macintosh HD *249.8 GB disk1
Logical Volume on disk0s2
D438A9A0-0FF5-41F0-BB46-4E9A41133503
Unencrypted
/dev/disk2
#: TYPE NAME SIZE IDENTIFIER
0: FDisk_partition_scheme *7.8 GB disk2
1: DOS_FAT_32 NO NAME 7.8 GB disk2s1
```

4. U盘为/dev/disk2，取消U盘挂载
```sehll
> diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
```
5. 将Centos系统写入U盘
```shell
> sudo dd if=~/Downloads/CentOS-7-x86_64-Minimal-1503-01.img.dmg of=/dev/disk2 bs=1m
```

6. 取出u盘插入Mac Mini系统，开机按住 option(alt) 键（必须在出现苹果图标前按住），这时候能看到U盘系统，选择U盘启动，安装linux时候格式化硬盘全盘。

ps:
> Centos 7安装lxc需要先安装EPEL源，默认的yum源没有lxc
Mac Mini需要恢复OSX系统，需要通过网络安装恢复，方法开机按 alt键，到选择界面 按cmd + r即可，非常容易
