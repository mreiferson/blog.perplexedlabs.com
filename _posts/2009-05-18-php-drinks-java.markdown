---
date: '2009-05-18 11:46:39'
layout: post
slug: php-drinks-java
status: publish
title: PHP Drinks Java
wordpress_id: '267'
categories:
- Development
- PHP
tags:
- java
- php
---

**[This](http://www.travisswicegood.com/index.php/2009/05/13/magic-is-to-python-as-java-is-to-php)** post got me thinking about exactly why it is that PHP developers dislike Java?

While researching I stumbled upon **[this](http://phpadvent.org/2008/php-is-not-java-by-luke-welling)** post which attempts to explain why you shouldn't treat PHP as if it were Java.  The example singleton code looks nearly identical in both PHP and Java.  The author suggests that an "experienced" PHP developer wouldn't attempt to think in terms of Java and would instead choose to implement this concept in PHP with the use of a global variable.  This is ridiculous.  It's crap like this that gives PHP a bad rap.

No, **PHP isn't Java** and shouldn't be treated as such.  Nor should you abandon all logic and reason and disregard features of the language that make it _appear_ Java-like.

PHP's object model borrows heavily from Java.  This is fact.  Its property and method visibility, single inheritance, interfaces, final classes, and abstractions are all very Java-like.  Not to mention exceptions and garbage collection.  I'll be damned if that doesn't cover a large portion of the features that developers use to write software in both PHP and Java.  Are all of these concepts exclusive to Java?  No, they're not.  It just seems that the PHP team felt that Java made some good decisions and decided to emulate them.

I'll bet that lots of PHP code is unknowingly written Java-like.  Is this because a PHP developer is "thinking in Java"?  Perhaps it's because the foundational toolset which a PHP developer was given was designed with Java in mind.  I'm not sure it makes sense to decide to use PHP to solve a problem and then fight with/ignore/don't take advantage of the features of the language.  I think that would be a good reason to choose another language, no?

How could a language (obviously) borrow so heavily from another language and simultaneously shun the mention of that language?
