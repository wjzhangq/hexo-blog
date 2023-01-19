---
title: Cross-Origin Resource Sharing 实现ajax的跨域请求
date: 2010-11-23 23:28:39
tags:
    - PHP
---

转载:http://blog.csdn.net/net_lover/archive/2010/01/11/5172509.aspx   
Cross-Origin Resource Sharing 跨域资源共享，该规范地址：http://www.w3.org/TR/access-control/和http://dev.w3.org/2006/waf/access-control/  
该规范提供了一种更安全的跨域数据交换方法。具体规范的介绍可以访问上面提供的网站地址。值得注意的是：该规范只能应用在类似 XMLHttprequest 这样的 API 容器内。IE8、Firefox 3.5 及其以后的版本、Chrome浏览器、Safari 4 等已经实现了 Cross-Origin Resource Sharing 规范，已经可以进行跨域请求了。  
最新版本的浏览器中,跨域ajax请求时候,浏览也会向服务器发送请求,当服务器响应头部设置了Access-Control-Allow-Origin项,那么ajax请求能正确完成.

Request Header:
```
Origin:http://localhost
Referer:http://localhost/cross/cross.html
User-Agent:Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_5_8; en-US) AppleWebKit/534.3 (KHTML, like Gecko) Chrome/6.0.472.53 Safari/534.3
```

Response Headers:
```
Access-Control-Allow-Origin:http://localhost
Server:Apache/2.2.16 (Unix) mod_ssl/2.2.16 OpenSSL/1.0.0a DAV/2 PHP/5.3.3
X-Powered-By:PHP/5.3.3
```

php 代码
```php
<?php
header(“Access-Control-Allow-Origin: http://localhost”);
echo ‘cross is ok’;
?>
```