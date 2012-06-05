---
date: '2009-10-05 09:00:30'
layout: post
slug: php-custom-session-handler
status: publish
title: PHP Custom MySQL Session Handler
wordpress_id: '302'
categories:
- Development
- Infrastructure
- PHP
tags:
- memcache
- mysql
- php
- session
- session handler
---

### The Problem


I'm sure many have used PHP's default session handling capabilities.  By default, PHP uses the filesystem to store session data naming files with their session id # and putting them in /tmp.

This is done for the sake of simplicity.  On a single-server, low load website, this particular setup works fine.  It's when you start having multiple simultaneous requests from a single client (identified by a session) that the problems begin to show their ugly heads.  Utilizing AJAX multiple simultaneous requests might be the norm, even for a low load website.

Essentially, in order to prevent [race conditions](http://en.wikipedia.org/wiki/Race_condition) PHP internally uses a lock to maintain exclusive access to the file containing the session data for the client connection.  This means that as soon as a single request acquires exclusive access to that session file, no other request can access the file until the original request completes.  What happens when that second request asks to start the session?  It waits.


### The Solution


Fortunately there's a solution to all this.  Implementing your own custom session handler and moving your session storage backend to another technology (such as a MySQL database or memcache) affords you the ability to handle simultaneous requests in a thread-safe manner.  Remember, it's a good thing that PHP prevents race conditions by locking the session file.  What we're looking to do is increase the granularity of the lock to the level of individual session data key => value pairs.

For this post we're going to stick to storing sessions in the database with our own custom session save handler.  Perhaps in another post I'll talk about doing the same in memcache.  It's the theory we're concerned about, not necessarily the exact storage mechanism implementation.

Implementing your own custom session handler is simply a matter of calling session_set_save_handler() with the appropriate callback methods for handling the following scenarios:



	
  * **open**
Open function, this works like a constructor in classes and is executed when the session is being opened. The open function expects two parameters, where the first is the save path and the second is the session name.

	
  * **close**
Close function, this works like a destructor in classes and is executed when the session operation is done.

	
  * **read**
Read function must return string value always to make save handler work as expected. Return empty string if there is no data to read. Return values from other handlers are converted to boolean expression. TRUE for success, FALSE for failure.

	
  * **write**
The "write" handler is not executed until after the output stream is closed.

	
  * **destroy**
The destroy handler, this is executed when a session is destroyed with session_destroy() and takes the session id as its only parameter.

	
  * **gc**
The garbage collector, this is executed when the session garbage collector is executed and takes the max session lifetime as its only parameter.



There's a well known trick for situations like this that allow you to pass a class method (static or instance) for callbacks.  Let's take a look at a simple example.  This correctly implements the required methods but obviously doesn't do much:

```php
class MySession
{
	public function __construct()
	{
		session_set_save_handler(
						array('MySession', 'sess_open'), 
						array('MySession', 'sess_close'),
						array('MySession', 'sess_read'),
						array('MySession', 'sess_write'),
						array('MySession', 'sess_destroy'),
						array('MySession', 'sess_gc')
						);
		
		ini_set('session.auto_start',					0);
		ini_set('session.gc_probability',				1);
		ini_set('session.gc_divisor',					100);
		ini_set('session.gc_maxlifetime',				604800);
		ini_set('session.referer_check',				'');
		ini_set('session.entropy_file',					'/dev/urandom');
		ini_set('session.entropy_length',				16);
		ini_set('session.use_cookies',					1);
		ini_set('session.use_only_cookies',				1);
		ini_set('session.use_trans_sid',				0);
		ini_set('session.hash_function',				1);
		ini_seT('session.hash_bits_per_character',		5);
		
		session_cache_limiter('nocache');
		session_set_cookie_params(0, '/', '.mydomainname.com');
		session_name('mySessionName');
		session_start();
	}

	public static function sess_open($save_path, $session_name)
	{
		return true;
	}

	public static function sess_close()
	{
		return true;
	}

	public static function sess_read($id)
	{
		return '';
	}

	public static function sess_write($id, $sess_data)
	{
		return true;
	}

	public static function sess_destroy($id)
	{
		return true;
	}

	public static function sess_gc($maxlifetime)
	{
		return true;
	}
}
```



### SPL and Storing Session Data In MySQL



Adding support for MySQL to this class is fairly trivial.  Let's start off by creating a table to store our session data:

```sql
CREATE TABLE `sessions` (
  `sesskey` char(32) NOT NULL,
  `timestamp` timestamp NOT NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
  `varkey` varchar(128) NOT NULL,
  `varval` longtext NOT NULL,
  PRIMARY KEY  (`sesskey`,`varkey`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

[PHP's SPL](http://us3.php.net/spl) provides the ability to create objects that behave as though they were arrays.  You can make objects that iterate, respond to accessing them via array notation ($object['key']), and lots of other interesting things.

We're going to enhance the MySession class with a couple SPL interfaces that will allow the object, when instantiated, to behave like an array.   We can then override the $_SESSION superglobal with an instance of our new MySession class.  It will provide an identical interface to access session data while internally storing session data to the database.  Also, internally it will use methods for row-level locking via MySQL's advisory locks (GET_LOCK() and RELEASE_LOCK()).

The next step is to implement the required methods for the interfaces we're using.  These methods allow the object to behave as though it's an array.

```php
class MySession implements Countable, ArrayAccess, Iterator
{
	private $index;
	private $curElement;
	private $locks = array();
	private $sessionName = 'sessionId';
	private $serialize = 'serialize';
	private $unserialize = 'unserialize';
	private $session_id = null;

	public function __construct()
	{
		session_set_save_handler(
						array('MySession', 'sess_open'), 
						array('MySession', 'sess_close'),
						array('MySession', 'sess_read'),
						array('MySession', 'sess_write'),
						array('MySession', 'sess_destroy'),
						array('MySession', 'sess_gc')
						);
		
		ini_set('session.auto_start',					0);
		ini_set('session.gc_probability',				1);
		ini_set('session.gc_divisor',					100);
		ini_set('session.gc_maxlifetime',				604800);
		ini_set('session.referer_check',				'');
		ini_set('session.entropy_file',					'/dev/urandom');
		ini_set('session.entropy_length',				16);
		ini_set('session.use_cookies',					1);
		ini_set('session.use_only_cookies',				1);
		ini_set('session.use_trans_sid',				0);
		ini_set('session.hash_function',				1);
		ini_seT('session.hash_bits_per_character',		5);
		
		session_cache_limiter('nocache');
		session_set_cookie_params(0, '/', '.mydomainname.com');
		session_name('mySessionName');
		session_start();
	}

	public function destroy()
	{
		$sessionName = session_name();
		$cookieInfo = session_get_cookie_params();
		$cookieExpires = time() - 3600;
		if((empty($cookieInfo['domain'])) && (empty($cookieInfo['secure']))) {
			setcookie($sessionName, '', $cookieExpires, $cookieInfo['path']);
		} elseif(empty($cookieInfo['secure'])) {
			setcookie($sessionName, '', $cookieExpires, $cookieInfo['path'], $cookieInfo['domain']);
		} else {
			setcookie($sessionName, '', $cookieExpires, $cookieInfo['path'], $cookieInfo['domain'], $cookieInfo['secure']);
		}
		unset($_COOKIE[$sessionName]);
		
		$dbo = DBO::getInstance();
		$q = "DELETE FROM `sessions` WHERE `sesskey` = '".$this->session_id."'";
		$dbo->query($q);
		
		session_destroy();
	}

	private function lockName($k)
	{
		return 'sesslock'.$this->session_id.$k;
	}

	public function locked($k)
	{
		$k = $this->lockName($k);
		
		return isset($this->locks[$k]);
	}

	public function acquire($k, $timeout = 0)
	{
		$k = $this->lockName($k);
		
		if(!isset($this->locks[$k])) {
			$dbo = DBO::getInstance();
			$q = "SELECT GET_LOCK('".$k."', ".$timeout.")";
			$rs = $dbo->query($q);
			$this->locks[$k] = $dbo->result($rs, 0);
			$dbo->fr($rs);
			
			return $this->locks[$k];
		}
		
		return false;
	}

	public function release($k)
	{
		$k = $this->lockName($k);
		
		unset($this->locks[$k]);
		
		$dbo = DBO::getInstance();
		$q = "SELECT RELEASE_LOCK('".$k."')";
		$rs = $dbo->query($q);
		$ret = $dbo->fetch($rs);
		$dbo->fr($rs);

		return true;
	}

	public function count()
	{
		$dbo = DBO::getInstance();
		$q = "SELECT COUNT(*) FROM `sessions` WHERE `sesskey` = '".$this->session_id."'";
		$rs = $dbo->query($q);
		$ret = $dbo->result($rs, 0);
		$dbo->fr($rs);
		
		return $ret;
	}

	public function rewind()
	{
		$this->index = 0;
		$this->getCurElement();
	}

	private function getCurElement()
	{
		$dbo = DBO::getInstance();
		$q = "SELECT `varkey`, `varval` FROM `sessions` WHERE `sesskey` = '".$this->session_id."' LIMIT ".$this->index.",1";
		$rs = $dbo->query($q);
		$row = $dbo->fetch($rs);
		$dbo->fr($rs);
		if(is_array($row) && (count($row) == 2)) {
			$this->curElement = $row;
		} else {
			$this->curElement = array(null, null);
		}
	}

	public function key()
	{
		return $this->curElement[0];
	}

	public function current()
	{
		return call_user_func($this->unserialize, $this->curElement[1]);
	}

	public function next()
	{
		$this->index++;
		$this->getCurElement();
	}

	public function valid()
	{
		return ($this->curElement[0] !== null);
	}

	public function offsetSet($k, $v)
	{
		$dbo = DBO::getInstance();
		$q = "REPLACE INTO `sessions` (`sesskey`, `varkey`, `varval`) VALUES ('".$this->session_id."', '".$k."', '".$dbo->sanitize(call_user_func($this->serialize, $v))."')";
		$dbo->query($q);
	}

	public function offsetGet($k)
	{
		$dbo = DBO::getInstance();
		$q = "SELECT `varval` FROM `sessions` WHERE `sesskey` = '".$this->session_id."' AND `varkey` = '".$k."'";
		$rs = $dbo->query($q);
		if($ret = $dbo->result($rs, 0)) {
			$ret = call_user_func($this->unserialize, $ret);
		}
		$dbo->fr($rs);
		
		return $ret;
	}

	public function offsetUnset($k)
	{
		$dbo = DBO::getInstance();
		$q = "DELETE FROM `sessions` WHERE `sesskey` = '".$this->session_id."' AND `varkey` = '".$k."'";
		$dbo->query($q);
	}

	public function offsetExists($k)
	{
		$dbo = DBO::getInstance();
		$q = "SELECT `varval` FROM `sessions` WHERE `sesskey` = '".$this->session_id."' AND `varkey` = '".$k."'";
		$rs = $dbo->query($q);
		$ret = $dbo->result($rs, 0);
		$dbo->fr($rs);

		return (bool)$ret;
	}

	public static function sess_open($save_path, $session_name)
	{
		return true;
	}

	public static function sess_close()
	{
		return true;
	}

	public static function sess_read($id)
	{
		return '';
	}

	public static function sess_write($id, $sess_data)
	{
		return true;
	}

	public static function sess_destroy($id)
	{
		return true;
	}

	public static function sess_gc($maxlifetime)
	{
		$dbo = DBO::getInstance();
		$q = "DELETE FROM `sessions` WHERE `timestamp` < '".date('Y-m-d H:i:s', time() - $maxlifetime)."'";
		$dbo->query($q);
		
		return $dbo->query($q);
	}
}
```

The $dbo object is just an example of an interface to the database through the use of a singleton.  Replace the $dbo object with your preferred mysql database interface and you'll be set to go!

Starting your session is now as simple as:

```php
$_SESSION = new MySession;
``` 
