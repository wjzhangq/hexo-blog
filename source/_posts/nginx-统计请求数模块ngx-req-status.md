---
title: nginx 统计请求数模块ngx_req_status
date: 2015-10-12 12:08:14
tags:
    - LINUX
    - NGINX
---

公司服务器需要监控网站页面请求。  
使用nginx的ngx_req_status模块刚好能满足需求。
请求结果如下：  
```
zone_name key max_active max_bw traffic requests active bandwidth
server_name localhost 2 0 4160 4 1 0
server_url localhost/ 2 0 1432 2 0 0
server_url localhost/req-status 1 0 2728 2 1 0
```
可以显示每个url pv, 活跃连接数等等。  
安装方法如下：
```
wget http://nginx.org/download/nginx-1.8.0.tar.gz
git clone https://github.com/zls0424/ngx_req_status.git
tar -zxvf nginx-1.8.0.tar.gz 
cd nginx-1.8.0
patch -p1 < /home/zhangwenjin/src/ngx_req_status/write_filter-1.7.11.patch
./configure --prefix=/usr/local/nginx --add-module=/home/zhangwenjin/src/ngx_req_status
make -j2
sudo make install
```
nginx配置如下：  
http配置：  
```
req_status_zone server_name $server_name 256k;
req_status_zone server_url  $server_name$request_uri$query_string 256k;
req_status server_name  server_url;
```
server配置：
```
location /req-status {
    req_status_show on;
}
```