---
date: '2008-02-08 10:52:59'
layout: post
slug: php-realtime-output-to-browser
status: publish
title: PHP Realtime Output to Browser
wordpress_id: '26'
categories:
- Development
- PHP
tags:
- ob_flush
- ob_implicit_flush
- php
---

I write a lot of administrative scripts that have rather long execution times.  Often it's desirable to be able to monitor the progress of the script in the browser, in realtime.  By default I have my PHP installation setup with gzip output buffering enabled.  This reduces the size of data transmitted to the client but also (obviously) buffers the output.

The PHP manual states that calling **[ob_implicit_flush();](http://us3.php.net/manual/en/function.ob-implicit-flush.php)** _"...will turn implicit flushing on or off. Implicit flushing will result in a flush operation after every output call, so that explicit calls to flush() will no longer be needed."_.  In my experience this doesn't actually happen.

The answer is simple.  After every portion of output (whether it be nested html, echo, print - whatever) simply call **[ob_flush();](http://us3.php.net/manual/en/function.ob-flush.php)**.
