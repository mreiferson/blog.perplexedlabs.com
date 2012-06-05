---
date: '2010-03-02 08:00:10'
layout: post
slug: php-forking-to-concurrency
status: publish
title: PHP Forking to Concurrency with pcntl_fork()
wordpress_id: '376'
categories:
- Development
- PHP
tags:
- concurrency
- fork
- parallel
- pcntl
- pcntl_fork
- pcntl_wait
- php
- process
- thread
- unix
---

I find it interesting and challenging to bend PHP in ways it probably shouldn't be bent.  Almost always I walk away pleasantly surprised at it's ability to solve a variety of problems.

Consider this example.  Let's say you want to take advantage of more than one core for a given process.  Perhaps it performs many intensive computations and on a single core would take an hour to run.  Since a PHP process is single threaded you won't optimally take advantage of the available multi-core resources you may have.

Fortunately, via the Process Control ([PCNTL](http://php.net/manual/en/book.pcntl.php)) extension, PHP provides a way to fork new child processes.  Forking is the concept of duplicating a thread of execution from the parent to a new child.  [pcntl_fork()](http://www.php.net/manual/en/function.pcntl-fork.php) is the function that does this.

The framework for using this extension is as follows:

```php
$maxChildren = 4;
$numChildren = 0;
foreach($unitsOfWork as $unit) {
	$pids[$numChildren] = pcntl_fork();
	if(!$pids[$numChildren]) {
		// do work
		doWork($unit);
		posix_kill(getmypid(), 9);
	} else {
		$numChildren++;
		if($numChildren == $maxChildren) {
			pcntl_wait($status);
			$numChildren--;
		}
	}
}
```

When a new child is forked via pcntl_fork() the pid is returned.  The if statement following the fork allows the child and parent to split their flow of execution based on who they are (i.e. the child does the work and kills itself - the parent tests for hitting the max number of children and waits, otherwise it creates another child).  The pcntl_wait() function is called when we hit $maxChildren, it blocks until a child exits.

Remember, if you want use database connections in your children, they each need to initialize their own connection.  Resources such as database connections are not thread safe.
