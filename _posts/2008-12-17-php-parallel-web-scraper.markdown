---
date: '2008-12-17 15:56:00'
layout: post
slug: php-parallel-web-scraper
status: publish
title: PHP Parallel Web Scraper
wordpress_id: '90'
categories:
- Development
- PHP
tags:
- curl
- parallel
- php
- scrape
---

Data is the most fundamental component of today's web applications.  Scraping and combining data from multiple sources to enhance, re-calculate, and re-display is an everyday occurance.

Scraping a list of URLs asynchronously is just about the slowest possible way to do it.  Fortunately, PHP 5.2+, via its cURL multi_* functions, gives us a way of downloading the data in parallel.  These functions are poorly documented compared to much of the PHP standard library, however, the process is still fairly straightforward.

What I attempted to do was abstract away the fundamental aspects of grouping and retrieving a large list of URLs to scrape.  The usage is simple:

```php
$list = array('MSFT', 'GOOG', 'YHOO', 'INTC', 'AAPL', 
    'CSCO', 'C', 'CEDC', 'IBM', 'ORCL', 'SAP', 'CA');

function urlFunc($data) {
   return 'http://finance.yahoo.com/q/ks?s='.$data;
}

function processFunc($k, $data) {
   echo 'processing html for '.$k."\n";
}

function dbFunc($k, $data) {
   echo 'storing scraped data to db for '.$k."\n";
}

Scraper::scrape($list, 4, 'urlFunc', 'processFunc', 'dbFunc');
```

Note: the argument list, after the 3rd parameter, is dynamic... ie. you can add any number of functions to otherwise process, manipulate, or store the data.  They will be called sequentially and passed the return value of the previous call.

Here is the class listing:

```php
class Scraper
{
	private $curlOptions = null;
	
	/**
	 * Wrapper to scrape a generic list of items, in groups, in parallel
	 *
	 * Argument list is dynamic, functions are called sequentially...
	 * ie. $urlList is divided into groups of $groupSize urls, each url is passed to
	 * the first function specificed in the dynamic arguments.  The data returned is then
	 * passed to the next function specified, and so on...
	 *
	 * @access public
	 * @param array $list array of data
	 * @param int $groupSize size of chunk to process in parallel
	 * @return bool whether the operation was successful
	 */
	static public function scrape($list, $groupSize = 10, $urlFunc = null)
	{
		$args = func_get_args();
		$funcs = array_slice($args, 3);
		
		if(!is_array($list) || !count($list) || empty($groupSize)) {
			return false;
		}
		
		$group = array();
		$c = 0;
		$i = 0;
		$total = count($list);
		foreach($list as $k => $v) {
			if(!empty($urlFunc) &amp;&amp; is_callable($urlFunc)) {
				$v = call_user_func_array($urlFunc, array($v));
			}
			$group[$k] = $v;
			$c++;
			$i++;
			if(($c == $groupSize) || ($i == $total)) {
				self::getMulti($group, $funcs);
				$c = 0;
				$group = array();
			}
		}
		
		return true;
	}
	
	/**
	 * Performs the parallel retrieval of an arbitrary list of urls
	 *
	 * Passed funcs are called sequentially, as requests complete and 
	 * data is available, with the return value of the previous
	 * function call...
	 *
	 * @access private
	 * @param array $urls array of URLs
	 * @param array $funcs array of functions to call as data returns
	 */
	static private function getMulti($urls, $funcs = array())
	{
		$curl = array();
		$multi = curl_multi_init();
		foreach($urls as $k => $v) {
			$curl[$k] = curl_init();
			
			curl_setopt($curl[$k], CURLOPT_URL, $v);
			curl_setopt($curl[$k], CURLOPT_RETURNTRANSFER, true);
		
			if(!empty(self::$curlOptions)) {
				curl_setopt_array($curl[$k], self::$curlOptions);
			}
			
			curl_multi_add_handle($multi, $curl[$k]);
		}
		
		$running = null;
		do {
			curl_multi_exec($multi, $running);
			while(($info = curl_multi_info_read($multi)) !== false) {
				$key = array_search($info['handle'], $curl, true);
				$return = curl_multi_getcontent($info['handle']);
				curl_multi_remove_handle($multi, $info['handle']);
				foreach($funcs as $func) {
					$return = call_user_func_array($func, array($key, $return));
				}
			}
		} while($running > 0);

		curl_multi_close($multi);
	}
}
``` 
