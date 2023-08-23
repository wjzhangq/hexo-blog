---
title: Mac系统终端sudo免密码
date: 2023-08-23 17:53:54
tags:
    - MAC
---

### 问题描述
经常需要使用sudo命令，每次都需要输入密码，很麻烦。  
使用修改/etc/sudoers文件，添加NOPASSWD参数，可以实现免密码。

### 操作步骤
1. 打开终端，输入命令：
```shell
	
sudo chmod u+w /etc/sudoers
sudo visudo
sudo chmod u+w /etc/sudoers
```

2. 修改过内容
```shell
 %admin ALL=(ALL) NOPASSWD: ALL
```