---
title: Phalcon 学习
date: 2015-09-17 12:02:39
tags:
---
Phalcon 是开源、全功能栈、使用C 扩展编写、针对高性能优化的PHP 5 框架  
### Phalcon安装：
```
> git clone --depth=1 git://github.com/phalcon/cphalcon.git
> cd cphalcon/build
> sudo ./install
add "extension=phalcon.so" to phalcon.ini
```
安裝phalcon dev tool
```
> git clone https://github.com/phalcon/phalcon-devtools
> ./phalcon-devtools/phalcon.sh
> ln -s ~/phalcon-devtools/phalcon.php /usr/bin/phalcon
> chmod ugo+x /usr/bin/phalcon
```
创建项目
```
> phalcon project test
```