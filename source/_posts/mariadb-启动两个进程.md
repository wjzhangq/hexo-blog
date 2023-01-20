---
title: mariadb 启动两个进程
date: 2015-07-28 11:54:39
tags:
    - LINUX
---

业务需要在一台机器上启动两个mysql进程。  
解决方案有两种：  
1. 方案一：将mysql启动脚本copy一份，指向不同目录
2. 方案二：使用mysqld_multi 管理启动多个实例。

最终我选用方案一。  

### 方案一：

1. 创建新数据文件目录：
```shell
> mkdir -p /opt/mysql/jira_db_4406
> chown -R mysql:mysql /opt/mysql/mysql
```
2. 初始化目录
```shell
> mysql_install_db --datadir=/opt/mysql/jira_db_4406 --user=mysql
> cp /etc/my.cnf /opt/mysql/jira_db_4406
> cp /etc/init.d/mysql /etc/init.d/mysql_jira
```
3. 修改/opt/mysql/jira_db_4406/my.cnf 中端口号
4. 修改启动脚本 /etc/init.d/mysql_jira
```shell
$bindir/mysqld_safe --defaults-file=/opt/mysql/jira_db_4406/my.cnf --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null 2>&1
```
> (ps:重点加–defaults-file=/opt/mysql/jira_db_4406/my.cnf –defaults-file 一定要是第一个参数)
5. 启动
```
> /etc/init.d/mysql_jira start
> chkconfig --level 3 mysql_jira on
```
5. 修改默认密码
```
> mysqladmin -u root password 123456
```
6. 支持远程登录
```
> mysql -h 127.0.0.1 -P 4406 -u root -p
mysql > GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql > flush privileges;
```
