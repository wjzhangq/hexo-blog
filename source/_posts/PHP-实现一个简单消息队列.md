---
title: PHP 实现一个简单消息队列
date: 2022-06-27 21:44:26
tags:
    - PHP
---
直接上代码。

```php
<?php
/**
 * 任务列表
 *
 */

/**
CREATE TABLE `t_job`  (
`job_id` int NOT NULL AUTO_INCREMENT COMMENT '任务id',
`at` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '任务执行时间',
`status` tinyint NOT NULL DEFAULT 0 COMMENT '任务状态，0 默认 1 执行中 2执行成功 -2 执行失败',
`lock_date` timestamp NULL COMMENT '任务锁定时间',
`cost` float NULL COMMENT '任务执行耗时',
`name` varchar(32) NOT NULL COMMENT '任务名称',
`params` text NULL COMMENT '任务参数',
`retry_cnt` int NULL COMMENT '任务重试次数',
PRIMARY KEY (`job_id`)
) COMMENT = '任务信息';
 */

class JobQueue implements Iterator
{
    static $is_stop = false;
    private $table_name = "t_job";
    private $SLEEP_TIME = 2; // 如果没有数据，间隔1s获取

    private $job_id = 0; //任务id
    private $start_time = 0; //任务时间
    private $err = "";

    /**
     * 数据库名称
     */
    public function __construct($table_name = "")
    {
        if ($table_name) {
            $this->table_name = $table_name;
        }

    }

    /**
     * first start, log start
     */
    public function rewind(): void
    {
        $this->log("start loop");
    }

    /**
     * return data, method, data
     *
     */
    public function current()
    {
        $db = di_get("db");
        $sql = sprintf('SELECT *  FROM `%s` WHERE `job_id`=%s', $this->table_name, $this->job_id);
        $row = $db->getRow($sql);
        $row['params'] = json_decode($row['params'], true); //解码
        $this->start_time = $this->microtime_float();
        $this->err = "";
        return $row;
    }

    /**
     * return key, after current
     */
    public function key()
    {
        return $this->job_id;
    }

    /**
     * 确认上一个任务完成，并记录时间和结果，如果需要重试，也这里处理
     */
    public function next(): void
    {
        $db = di_get("db");
        $cost = $this->microtime_float() - $this->start_time; //执行时间
        if (empty($this->err)) {
            $status = 2; //执行成功
            $sql = sprintf("UPDATE `%s` SET cost=%s, status=2 WHERE `job_id`=%s", $this->table_name, $cost, $this->job_id);
        } else {
            if (is_string($this->err)) {
                $status = -2;
                $msg = $this->err;
            } else {
                $status = 2; //执行成功
                $msg = json_encode($this->err);

            }
            $sql = sprintf("UPDATE `%s` SET `cost`=%s, `status`=%s, `msg`='%s' WHERE `job_id`=%s", $this->table_name, $cost, $status, $msg, $this->job_id);

        }
        $db->query($sql);
    }

    /**
     * 获取一个项目，如果没有等等
     */
    public function valid(): bool
    {
        $db = di_get("db");
        while (true) {
            $sql = sprintf("SELECT `job_id` FROM `%s` WHERE status=0 AND `at`<=current_timestamp ORDER BY `job_id` ASC LIMIT 1", $this->table_name);
            $row = $db->getRow($sql);

            if (!$row) {
                sleep($this->SLEEP_TIME);
                continue; //等待后看是否有消息
            }

            $job_id = $row['job_id'];
            //lock
            $sql = sprintf("UPDATE %s SET status=1, lock_date=current_timestamp WHERE status=0 AND `job_id`=%s", $this->table_name, $row['job_id']);
            $sth = $db->query($sql);
            $n = $sth->rowCount();
            if ($n == 0) {
                continue; //当前记录已经被其他进程锁定，重新获取
            }
            $this->job_id = $job_id;
            return true; //开始获取数据
        }
    }

    public function log($data)
    {
        $buf = $data;
        if (!is_string($data)) {
            $buf = json_encode($data);
        }

        $debug_info = debug_backtrace();
        //var_dump($debug_info);
        $file_path = $debug_info[0]['file'];
        $line_no = $debug_info[0]['line'];
        //$file_path = substr($file_path, strlen(APP_PATH));

        $buf = sprintf("%s %s:%s %s\n", date("H:i:s"), $file_path, $line_no, $buf);

        echo ($buf);
    }

    /**
     * 获取时间戳
     */
    public function microtime_float()
    {
        list($usec, $sec) = explode(" ", microtime());
        return ((float) $usec + (float) $sec);
    }

    /**
     * 增加任务
     * @params afert 多久后执行,单位s
     */
    public function add($name, $params, int $afert = 0)
    {
        $db = di_get("db");
        $buf = json_encode($params);

        if ($afert > 0) {
            $sql = sprintf("INSERT INTO `%s` SET `at`= date_add(current_timestamp, interval %s second) , `name` = '%s', `params`='%s'", $this->table_name, $afert, $name, $buf);
        } else {
            $sql = sprintf("INSERT INTO `%s` SET `at`= current_timestamp,  `name` = '%s', `params`='%s'", $this->table_name, $name, $buf);
        }
        $db->query($sql);
        $job_id = $db->lastInsertId();

        return $job_id;
    }

    /**
     * 增加唯一任务，如果当前任务已经存在，不在添加
     */
    public function add_unique($name, $params)
    {
        $db = di_get("db");

        $sql = sprintf("SELECT name FROM `%s` WHERE `status` = 0 AND `name` ='%s'", $this->table_name, $name);
        $row = $db->getRow($sql);
        if ($row) {
            return 0; //不在添加
        }

        return $this->add($name, $params, 0);
    }

    /**
     * 确认任务信息
     */
    public function done($err = "")
    {
        $this->err = $err;
    }

    /**
     * 重新
     */
    public function retry(int $after = 0)
    {
        $db = di_get("db");
        if ($after > 0) {
            $sql = sprintf("UPDATE `%s` SET `at`=date_add(current_timestamp, interval %s second), `status`=0, `retry_cnt` =`retry_cnt` + 1  WHERE `job_id`=%s", $this->table_name, $after, $this->job_id);
        } else {
            $sql = sprintf("UPDATE `%s` SET `at`=current_timestamp, `status`=0, `retry_cnt` =`retry_cnt` + 1  WHERE `job_id`=%s", $this->table_name, $this->job_id);
        }
        $db->query($sql);

    }

    /**
     * 获取未完成的
     */
    public function count()
    {
        $db = di_get("db");

        $sql = sprintf("SELECT COUNT(*) FROM `%s` WHERE status=0", $this->table_name);
        return intval($db->getOne($sql));
    }
}

```