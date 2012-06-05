---
date: '2008-02-25 12:57:33'
layout: post
slug: php-garbage-collection-and-memory-leaks
status: publish
title: PHP garbage collection and memory leaks
wordpress_id: '32'
categories:
- Development
- PHP
tags:
- garbage collection
- memory leak
- php
---

Been working on a command line script that takes days to finish execution (yes, DAYS).  Unfortunately, it had a healthy flow of memory leaking.  It would fatally crash with your typical "out of memory" error.

I spent hours debugging the potential source - free'd results of mysql queries, unset variables, you name it I tried it.  I kind of gave up and instead chose to add in a "start at" parameter so that I could just re-execute and jump back in to the loop where it crashed last time.

Today, through natural processes of improving functionality and cleaning code, I moved the block of code that was leaking memory into a function so that it could be recycled elsewhere.  **By simply moving the code into a function I suspect PHP now garbage collects after each call and memory no longer hemorrhages from its veins.**

I hope this helps someone else out there!
