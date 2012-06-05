---
date: '2010-11-01 18:50:26'
layout: post
slug: asynchronous-dns-resolution-in-tornados-asynchttpclient-curl-multi-c-ares
status: publish
title: Async DNS Resolution in Tornado's AsyncHttpClient (curl multi, c-ares)
wordpress_id: '513'
categories:
- Development
- Infrastructure
tags:
- async
- asynchronous
- asynchttpclient
- c-ares
- curl
- dns
- libcurl
- Python
- tornado
---

I learned some rather important facts about cURL's multi interface (which makes it possible to perform asynchronous HTTP requests and what Python's [Tornado](http://www.tornadoweb.org/) framework uses under the hood in it's AsyncHttpClient helper).

I was investigating some intermittent issues in an application at work - transient DNS issues were causing the application to become unresponsive.  This was confusing at first because it was written to perform HTTP requests asynchronously.  The important thing here is that it was specifically DNS resolution that was failing.  As I dug deeper I realized that simple health checks, ones that did not perform any HTTP requests, were also hanging... something had to be blocking.

Taking a look at the Tornado source, unsurprisingly, it was leveraging the [multi](http://curl.haxx.se/libcurl/c/libcurl-multi.html) functionality of libcurl (via pycurl).  What was surprising to me was that the DNS resolution portion of the multi interface, by default, blocks on non-windows installations.  Only when compiled with [c-ares](http://c-ares.haxx.se/) support does it perform these async.

You learn something new every day!
