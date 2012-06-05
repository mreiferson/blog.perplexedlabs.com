---
date: '2010-03-04 08:00:08'
layout: post
slug: python-data-sharing-in-the-multiprocessing-module
status: publish
title: Python data sharing in the multiprocessing module
wordpress_id: '430'
categories:
- Development
- Python
tags:
- multiprocessing
- Python
---

Python's [multiprocessing](http://docs.python.org/library/multiprocessing.html) module is a great tool that abstracts the details of forking and managing child processes in an interface inspired by the [threading](http://docs.python.org/library/threading.html) module.  The benefit to using processes over threads is that you effectively avoid the issues of the GIL (Global Interpreter Lock).

I wanted to share my experience with sharing static data between the parent and the forked children.  The solution I ultimately went with is trivially implemented and works well.  It takes advantage of the fact that the children import the same modules of the parent.  If you house your data in a shared module, it's accessible in both places.

The directory structure looks like this:



> 

>     
>     
>     mypackage/
>         __init__.py
>         mp.py
>         myglobals.py
>     myscript.py
>     
> 
> 




Here's my light wrapper around the multiprocessing module, mp.py:

```python
import multiprocessing

import MySQLdb

import myglobals

# handles each unit of work, in this case a SQL query
def worker_do(sql):
    myglobals.cursor.execute(sql)

# called once upon worker initialization
def worker_init():
    myglobals.conn = MySQLdb.connect(**myglobals.config['db'])
    myglobals.cursor = myglobals.conn.cursor()
    myglobals.cursor.execute('SET AUTOCOMMIT=1')

# wrapper for multiprocessing module
def do_work(queue, num_processes):
    pool = multiprocessing.Pool(num_processes, initializer=worker_init)
    pool.map(worker_do, queue, 1)
    pool.close()
    pool.join()
```

And here's my example script, myscript.py:

```python
import os
import sys

import mp
import myglobals

def main():
   # anything in the myglobals module will be accessible by the child processes
   # we could then programatically retrieve this config info from a file 
   # via ConfigParser
   #
   # for simplicity I hard-coded it here 
   myglobals.config = {
      'db': {
         'host': 'db1',
         'user': 'dbuser',
         'passwd': 'dbpasswd',
         'db': 'dbase' 
      }
   }
   
   # build a whole bunch of queries to perform via the workers
   queries = build_queries()
   
   # perform the multiprocessing operation
   mp.do_work(queries, 4)

   return 0


if __name__ == '__main__':
   sys.exit(main())
```

In this example the benefit would be to keep your database configuration code DRY - and share that data with the child processes.
