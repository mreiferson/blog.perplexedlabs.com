---
date: '2008-02-06 10:34:57'
layout: post
slug: mutex-with-php-and-mysql
status: publish
title: Mutex with PHP and mySQL
wordpress_id: '21'
categories:
- Development
- PHP
tags:
- mutex
- mysql
- php
---

There are certain situations where you want a periodically executed script to have one and only one instance running at a time to prevent against poisoning of data due to simultaneous execution.

There's actually an extremely simple way to accomplish this in a PHP environment by taking advantage of mySQL's built in "lock" functionality.

The following simple example illustrates the usage of the class that is listed below:
```php
isFree()) {
	$lock->lock();
	// execute
	$lock->release();
} else {
	trigger_error("ALREADY LOCKED... ABORTING!", E_USER_NOTICE);
}
?>
```

Class listing cLock.php: (keep in mind this class uses some helper functions of mine, particularly 'qdb()' and 'result()' which do the obvious)
```php
lockname = $name;
		$this->timeout = $timeout;
		$this->locked = -1;
	}
	
	function lock()
	{
		$rs = qdb("SELECT GET_LOCK('".$this->lockname."', ".$this->timeout.")");
		$this->locked = result($rs, 0);
		mysqli_free_result($rs);
	}
	
	function release()
	{
		$rs = qdb("SELECT RELEASE_LOCK('".$this->lockname."')");
		$this->locked = !result($rs, 0);
		mysqli_free_result($rs);
		
	}
	
	function isFree()
	{	
		$rs = qdb("SELECT IS_FREE_LOCK('".$this->lockname."')");
		$lock = (bool)result($rs, 0);
		mysqli_free_result($rs);
		
		return $lock;
	}
}

?>
``` 
