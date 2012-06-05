---
date: '2009-05-04 09:00:14'
layout: post
slug: php-jquery-ajax-javascript-long-polling
status: publish
title: PHP jQuery AJAX Javascript Long Polling
wordpress_id: '217'
categories:
- Development
- JavaScript
- PHP
tags:
- ajax
- JavaScript
- lighttpd
- long polling
- nginx
- php
---

### Background



"Long polling" is the name used to describe a technique which:





  * An AJAX request is made (utilizing a javascript framework such as jQuery)

  * The server waits for the data requested to be available, loops, and sleeps (your server-side PHP script)

  * This loop repeats after data is returned to the client and processed (usually in your AJAX request's onComplete callback function)


This essentially simulates a continuous real-time stream from the client to the server.  It can be more efficient than a regular polling technique because of the reduction in HTTP requests.  You're not asking over and over and over again for new data - you ask once and wait for an answer.  In most cases this reduces the latency in which data becomes available to your application.

There are a variety of use cases in which this technique can be handy.  At the top of the list are real-time web-based chat applications.  Each client executes a long polling loop for chat and user events (sign on/sign off/new message).  [Meebo](http://www.meebo.com) is perhaps the greatest example of this.

It's important to note some of the server side technical limitations of long polling.  Because connections remain open for considerably longer time than a typical HTTP request/response cycle you want your web server to be able to handle a large number of simultaneous connections.  Apache isn't the best candidate for this type of situation.  [nginx](http://nginx.net/) and [lighttpd](http://www.lighttpd.net/) are two lightweight web servers built from the ground up to handle a high volume of simultaneous connections.  Both support the FastCGI interface and as such can be configured to support PHP.  Again, Meebo uses lighttpd.

For similar reasons - it's also a good idea to choose a different sub-domain to handle long polling traffic.  Because of client side browser limitations you don't want long polling connections interfering with regular HTTP traffic delivering page and media resources for your application.



### Implementation



[jQuery](http://www.jquery.com) makes implementation a breeze.

```javascript
var lpOnComplete = function(response) {
	alert(response);
	// do more processing
	lpStart();
};

var lpStart = function() {
	$.post('/path/to/script', {}, lpOnComplete, 'json');
};

$(document).ready(lpStart);
```

Straightforward.  When the document is ready the loop begins.  Each iteration the returned data is processed and the loop is restarted.

On the server side - just like we discussed earlier:

```php
$time = time();
while((time() - $time) < 30) {
	// query memcache, database, etc. for new data
	$data = $datasource->getLatest();
	
	// if we have new data return it
	if(!empty($data)) {
		echo json_encode($data);
		break;
	}
	
	usleep(25000);
}
```

Actually, a couple points of interest here.  We don't actually loop _infinitely_ server side.  You may have noticed the logic for the while loop - if we've executed for more than 30 seconds we discontinue the loop and return nothing.  This nearly eliminates the possibility of substantial memory leaks.  Also, if we didn't put a cap on execution time we would need to print a "space" character and flush output buffers every iteration of the loop to keep PHP abreast to the status of this process/connection.  **Without output being sent PHP cannot determine if the connection was lost via connection_status() or connection_aborted()**.  As a result this could lead to a situation where there are an increasing number of "ghost" processes eating up server resources.  Not good!

That pretty much sums it up!  Not that difficult, right?

As always, questions/comments are welcome, hope this helps!


