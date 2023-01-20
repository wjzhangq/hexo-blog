---
title: Mysql 触发器
date: 2022-05-18 21:19:26
tags:
    - MYSQL
---

公司一个业务的Mysql user表总是莫名被修改。项目太老懒得去找具体原因了。  
最终觉得偷懒使用mysql触发器，将所有user表的改动记录到一个user_log表，如果用户信息被修改，通过查询user_log就能帮用户数据恢复为以前版本。

具体操作如下：  
1. 创建user_log 表
```mysql
CREATE TABLE `t_user_log` (
  `lid` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `account_id` varchar(64) DEFAULT NULL,
  `id_shows` text,
  `cdate` datetime DEFAULT NULL,
  PRIMARY KEY (`lid`),
  KEY `account_id` (`account_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4;
```

2. 创建user表insert触发器
```mysql
DELIMITER //
CREATE TRIGGER add_user
AFTER INSERT
ON t_users
FOR EACH ROW
    Insert into t_user_log(account_id, id_shows, cdate) VALUES(NEW.account_id, NEW.id_shows,NOW()) //
DELIMITER ;
```

3. 创建user表update触发器
```mysql
DELIMITER //
CREATE TRIGGER update_user
AFTER UPDATE
ON t_users
FOR EACH ROW
 BEGIN
    IF NEW.id_shows != OLD.id_shows THEN
        Insert into t_user_log(account_id, id_shows, cdate) VALUES(NEW.account_id, NEW.id_shows,NOW());
    END IF;
 END//
DELIMITER ;
```

> 查询所有触发器  
SHOW TRIGGERS;
