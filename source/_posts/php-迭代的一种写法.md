---
title: php 迭代的一种写法
date: 2011-02-22 11:47:52
tags:
    - PHP
---
```php
<?php
class Fib implements Iterator{
	var $a = 0;
	var $b = 1;
	var $i = 0;
	var $max = 100;

	public function __construct($n=100){
		$this->max = $n;
	}

	public function rewind(){
		$this->a = 0;
		$this->b = 1;
		$this->i = 0;
	}

	public function next(){
		$tmp = $this->a + $this->b;
		$this->a = $this->b;
		$this->b = $tmp;
		$this->i = $this->i + 1;
	}

	public function current(){
		return $this->b;
	}

	public function key(){
		return $this->i;
	}

	public function valid(){
		return $this->b < $this->max;
	}
}

$a = new Fib(100);
foreach($a as $k=>$v){
	echo sprintf("%d\t%d\n", $k, $v);
}

?>
```
git地址：https://gist.github.com/wjzhangq/838433