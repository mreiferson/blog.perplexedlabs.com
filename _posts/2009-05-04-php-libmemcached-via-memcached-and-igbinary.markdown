---
date: '2009-05-04 10:00:41'
layout: post
slug: php-libmemcached-via-memcached-and-igbinary
status: publish
title: PHP libmemcached via memcached and igbinary
wordpress_id: '215'
categories:
- Development
- PHP
tags:
- igbinary
- libmemcached
- memcache
- pecl
- php
---

Found some great PHP resources that I'd like to share.  I haven't seen much talk of these so I'm hoping I can help spread the word.

First off **[libmemcached](http://tangent.org/552/libmemcached.html)**.  

Most PHP folks are familiar with the **[memcache](http://pecl.php.net/package/memcache)** (note the lack of a 'd' in the name) PECL extension.  This extension exposes a simple API for PHP apps to interact with memcache instances.  It works - it's simple, stable, and has been available since 2004.  Nothing special.

On the other hand, **[libmemcached](http://tangent.org/552/libmemcached.html)** "is a small, thread-safe client library for the memcached protocol. The code has all been written with an eye to allow for both web and embedded usage."  - "It has been designed to be light on memory usage, thread safe, and provide full access to server side methods."  And, fortunately, there's a new PECL extension that wraps libmemcached in a client library for PHP called **[memcached](http://www.pecl.php.net/package/memcached)** (note the 'd').  It was released in late January and is still considered "beta" however in my testing it has been stable.  This extension provides a rich interface to your memcache instances including the new check and set (cas), replace, and append operations.  As libmemcached becomes more widely adopted and its development continues, it makes sense to unify support behind a common library.

Lastly, **[igbinary](http://opensource.dynamoid.com/)**.  This is a PHP extension which provides _binary_ serialization for PHP objects and data.  It's a drop in replacement for PHP's built in serializer.  Why is this important?  When storing data in memcache it is first serialized (this is done automatically by the client library, such as memcached).  Conversely when retrieving data from memcache the data is unserialized.  The default PHP serializer uses a textual representation of data and objects.  This is a waste of memory.   Also, as objects increase in size and complexity the time it takes to (un)serialize increases significantly.  Igbinary stores data in a compact binary format which reduces the memory footprint and performs operations faster.  Most importantly memcached has built in support to take advantage of igbinary as its default serializer, yet another reason to use it as your memcache client library.

Check these resources out and let me know how they work for you!
