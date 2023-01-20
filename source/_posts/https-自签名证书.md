---
title: https 自签名证书
date: 2015-10-16 12:11:08
tags:
---

公司需要将外网接口全部改为https方式，保证客户端数据传输的安全。并且将客户端连接改为长连接，这样能同时保证传输效率。客户端是自己的代码，所以使用自己的签名就行。  

### https自签名步骤：
```
openssl genrsa -des3 -out server.orig.key 2048 ＃生成密钥
openssl rsa -in server.orig.key -out server.key #去除密钥中的密码
openssl req -new -subj "/C=CN/ST=Beijing/L=Beijing/O=Beijing/OU=wefit/CN=*.wefit.com.cn" -key server.key -out server.csr #生成公钥
openssl x509 -req -days 36500 -in server.csr -signkey server.key -out server.crt #公钥签名生成证书
```
证书签名可选配置(可以不选，这里列出只是备注下)
```
[req]
req_extensions = v3_req

distinguished_name = req_distinguished_name

[req_distinguished_name]

[ v3_req ]

# Extensions to add to a certificate request

basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = *.wefit.com.cn
``` 

nginx 配置：
```
listen 443 ssl;
ssl_certificate /opt/ca/server.crt;
ssl_certificate_key /opt/ca/server.key;
```
一切ok。客户端长连接略过。强烈推荐python urllib3模块