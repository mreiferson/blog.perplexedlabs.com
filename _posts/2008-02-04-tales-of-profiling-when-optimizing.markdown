---
date: '2008-02-04 15:13:51'
layout: post
slug: tales-of-profiling-when-optimizing
status: publish
title: Tales of profiling when optimizing
wordpress_id: '16'
categories:
- Development
tags:
- optimizing
- php
- profiling
- sql
---

Spending most of the day optimizing a page that (when I began) took 8.6 seconds to execute.  Using some basic profiling techniques helped tremendously in not wasting my time and efforts attempting to optimize a portion of code that won't yield appreciable gains.

The time intensive code for applications I write generally fall into either data retrieval or data processing.  In this particular case the app was spending about 1.5 seconds to retrieve orders that composed a given trade - 1 query per order (p.s. there are 1000's and 1000's of orders).   I re-wrote this portion with 1 query to load into memory all the orders beforehand and cut execution time to 0.11 seconds.

After all is said and done, the page execution time sits at 1.7 seconds.   That's an ~80% improvement!
