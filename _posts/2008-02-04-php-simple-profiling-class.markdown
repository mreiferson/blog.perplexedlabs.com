---
date: '2008-02-04 13:47:58'
layout: post
slug: php-simple-profiling-class
status: publish
title: PHP Simple Profiling Class
wordpress_id: '15'
categories:
- Development
- PHP
tags:
- execution time
- microtime
- php
- profiling
---

Had to quickly get the execution time of certain segments of code so I whipped up this basic profiling class below.

Usage would go something like:

```php
start();
// ... long block of code ...
$et = $profiler->end();
echo 'longBlock et = '.$et.'  
';
?>
```

Class listing cProfiler.php:

```php
_i = 0;
		$this->stamps = array();
		$this->ets = array();
	}
	
	function __destruct() {}
	
	function _stamp() { return microtime(); }

	function start() {
		$this->stamps[$this->_i] = $this->_stamp();
		$this->_i++;
	}
	
	function getET($id = '') { return $this->ets[$id]; }
	
	function end($id = '')
	{
		$timeend = $this->_stamp();
		$et = false;
		if($this->_i > 0) {
			$timestart = $this->stamps[$this->_i - 1];
			unset($this->stamps[$this->_i - 1]);
			$this->_i--;
			$et = number_format(((substr($timeend,0,9)) + (substr($timeend,-10)) - (substr($timestart,0,9)) - (substr($timestart,-10))),4);
			$this->ets[$id] = $et;
		}
		
		return $et;
	}
}

?>
``` 
