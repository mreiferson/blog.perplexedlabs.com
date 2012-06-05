---
date: '2008-12-21 14:08:21'
layout: post
slug: php-sessions-on-localhost
status: publish
title: PHP Sessions on localhost
wordpress_id: '116'
categories:
- Development
- PHP
tags:
- localhost
- php
- sessions
---

Trying to setup a sandbox development environment on my laptop before traveling for the holidays.  Found [WAMP](http://www.wampserver.com/en/) which was a breeze to setup and get running.

Anyway, I ran into a problem where basic session code that my servers handled fine wasn't working as expected when accessed via localhost.   This is how I got things working:

Set ALL session configuration parameters BEFORE calling session_name() (which, additionally, should ALWAYS be called before session_start(), fyi).  This includes, most importantly, session cookie parameters.  The following is one example that would work for either localhost or operating on a production server:

```php
function domainName()
{
	$serverName = $_SERVER['SERVER_NAME'];
	$serverNameParts = explode('.', $serverName);
	if(count($serverNameParts) < 2) {
		return $serverName;
	} else {
		return $serverNameParts[count($serverNameParts) - 2].'.'.
			$serverNameParts[count($serverNameParts) - 1];
	}
}

function cookieDomainName()
{
	$domain = domainName();
		
	// if we're operating on localhost, provide a blank domain
	// otherwise cookies won't be set
	return ($domain == 'localhost') ? '' : '.'.$domain;
}

session_set_cookie_params(0, '/', cookieDomainName());
session_name('mySessionName');
session_start();
```

Hope this helps!
