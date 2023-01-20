---
title: DOCKER 打包PHP项目记录
date: 2022-12-01 23:40:55
tags:
    - PHP
---
整个包只有70M，包含mysql和redis。  
直接上dockerfile。
```docker
FROM php:7.4-fpm-alpine3.11
ADD repositories /etc/apk/repositories
ADD default.conf /
ADD index.html /
ADD run.sh /
RUN apk update && apk add nginx && \
    apk add m4 autoconf make gcc g++ linux-headers && \
    pecl install redis && docker-php-ext-enable redis && \
    docker-php-ext-install pdo_mysql opcache mysqli && \
    mkdir /run/nginx && \
    mv /default.conf /etc/nginx/conf.d && \
    touch /run/nginx/nginx.pid && \
    chmod 755 /run.sh && \
    apk del m4 autoconf make gcc g++ linux-headers

EXPOSE 80
EXPOSE 9000

ENTRYPOINT ["/run.sh"]
```