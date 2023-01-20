---
title: 博客升级为https
date: 2015-10-09 12:05:21
tags:
    - LINUX
    - SSL
---

网站升级为https  
最近比较流行https，我也准备吧自己博客升级为https。  

### 申请证书
申请证书参考 http://www.freehao123.com/nginx-startssl/  
StartSSL 网站提供免费的证书申请。  

### nginx配置
将startssl提供的.key文件和.crt文件拷贝到服务器   

.key文件每次nginx启动时候需要输入密码，因此需要先去掉.key文件的密码。方法如下：  
```shell
openssl rsa -in zhangwenjin.key -out zhangwenjin.key.unsecure
```
nginx 配置：
```
listen       443;
ssl on;
ssl_certificate /root/zhangwenjin.crt;
ssl_certificate_key /root/zhangwenjin.key.unsecure;
ssl_session_timeout 5m;
```
访问 https://blog.zhangwenjin.com 即可 

#### 开启OCSP Stapling
获取startssl根证书  
```
wget -O - https://www.startssl.com/certs/ca.pem https://www.startssl.com/certs/sub.class1.server.ca.pem | tee -a ca-certs.pem&gt; /dev/null
```
nginx 配置
```
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /home/www/zhangwenjin.com/crtca-certs.pem;
```