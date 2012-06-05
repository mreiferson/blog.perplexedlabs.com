---
date: '2009-11-15 15:37:38'
layout: post
slug: setup-python-2-6-4-mod_wsgi-2-6-and-django-1-1-1-on-centos-5-3-cpanel
status: publish
title: Setup Python 2.6.4, mod_wsgi 2.6, and Django 1.1.1 on CentOS 5.3 (cPanel)
wordpress_id: '378'
categories:
- Development
- Django
- Infrastructure
- Python
tags:
- centos
- Django
- linux
- mod_wsgi
- mysql
- mysql-python
- mysqldb
- Python
- setuptools
---

This is an update to my previous how-to [Setup Python 2.5, mod_wsgi, and Django 1.0 on CentOS 5 (cPanel)](http://blog.perplexedlabs.com/2008/11/10/setup-python-25-mod_wsgi-and-django-10-on-centos-5-cpanel/).

The biggest reason why I chose to go with Python 2.5 at the time was because the MySQL Python (MySQLdb) package didn't support Python 2.6.  The 1.2.3c1 release does so that roadblock is lifted.

The instructions are identical - nothing has really changed in that regard.  Just change the references from Python 2.5 to 2.6.  Here are the links to the versions I'm using successfully:



> 
Python 2.6.4: [http://www.python.org/ftp/python/2.6.4/Python-2.6.4.tgz](http://www.python.org/ftp/python/2.6.4/Python-2.6.4.tgz)

setuptools 0.6c11: [http://pypi.python.org/packages/2.6/s/setuptools/setuptools-0.6c11-py2.6.egg#md5=bfa92100bd772d5a213eedd356d64086](http://pypi.python.org/packages/2.6/s/setuptools/setuptools-0.6c11-py2.6.egg#md5=bfa92100bd772d5a213eedd356d64086)

MySQLdb 1.2.3c1: [http://sourceforge.net/projects/mysql-python/files/mysql-python-test/1.2.3c1/MySQL-python-1.2.3c1.tar.gz/download](http://sourceforge.net/projects/mysql-python/files/mysql-python-test/1.2.3c1/MySQL-python-1.2.3c1.tar.gz/download)

mod_wsgi 2.6: [http://modwsgi.googlecode.com/files/mod_wsgi-2.6.tar.gz](http://modwsgi.googlecode.com/files/mod_wsgi-2.6.tar.gz)

Django 1.1.1: [http://www.djangoproject.com/download/1.1.1/tarball/](http://www.djangoproject.com/download/1.1.1/tarball/)

