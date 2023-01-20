---
title: debian 离线安装包
date: 2014-04-27 02:27:13
tags:
    - LINUX
---

应用场景：  
很多相同机器需要安装同样的软件包，全部通过apt-get网络安装很不划算。  
可以考虑把同步服务器的安装包作为apt-get的离线源。

步骤：
```shell
#安装打包工具
apt-get install dpkg-dev
#生成离线目录
mkdir offlinePackage
#复制所有apt-get已有软件包
cp -r /var/cache/apt/archives  /offlinePackage
#更改文件权限
chmod 777 -R /offlinPackage/
#建立deb包依赖关系
dpkg-scanpackages /offlinePackage/ /dev/null |gzip >/offlinePackage/Packages.gz
#将生成的Packages.gz包复制到和deb同目录下
cp /offlinePackage/Packages.gz /offlinePackage/archives/Packages.gz
#打包快速传输
tar cvzf offlinePackage.tar.gz offlinePackage/
```
其他机器使用方式：
```shell
vim /etc/apt/sources.list
添加以下两行至文件末尾：
deb file:///offlinePackage archives/
deb-src file:///offlinePackage archives/
#将所有的其他deb全部注销掉（#）
apt-get update
sudo apt-get  install XXXXX
 ```