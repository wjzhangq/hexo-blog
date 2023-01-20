---
title: linux命令行发送smtp邮件
date: 2015-07-16 11:47:22
tags:
    - LINUX
---

服务器需要通过邮件发送统计数据。  
直接通过mail命令发送容易被当垃圾邮件。所以打算找一个靠谱的邮件服务商，通过smtp发送邮件就不会被当作垃圾邮件。  
mail stmp配置：
```shell
vim /etc/mail.rc
＃append
set from=xxx@zhangwenjin.com
set smtp=smtp.zhangwenjin.com
set smtp-auth-user=xxx@zhangwenjin.com
set smtp-auth-password=xxxx
```
发送方式：
```shell
echo "test" | mail -v -s 四川统计-`date +'%Y-%m-%d'` -c cc1@zhangwenjin.com -c cc2@zhangwenjin.com to@zhangwenjin.com
```
 
