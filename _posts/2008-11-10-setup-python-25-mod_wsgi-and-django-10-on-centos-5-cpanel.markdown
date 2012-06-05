---
date: '2008-11-10 10:53:56'
layout: post
slug: setup-python-25-mod_wsgi-and-django-10-on-centos-5-cpanel
status: publish
title: Setup Python 2.5, mod_wsgi, and Django 1.0 on CentOS 5 (cPanel)
wordpress_id: '57'
categories:
- CSS
- Development
- Django
- Infrastructure
- Python
tags:
- centos
- cpanel
- Django
- mod_wsgi
- Python
---

**Please read the update to this article [Setup Python 2.6.4, mod_wsgi 2.6, and Django 1.1.1 on CentOS 5.3 (cPanel)](http://blog.perplexedlabs.com/2009/11/15/setup-python-2-6-4-mod_wsgi-2-6-and-django-1-1-1-on-centos-5-3-cpanel/)**



### Installing Python 2.5 dependencies




#### Install sqlite3:




> $ wget http://www.sqlite.org/sqlite-amalgamation-3.6.4.tar.gz
$ tar xvfz sqlite-amalgamation-3.6.4
$ cd sqlite-amalgamation-3.6.4.tar.gz
$ ./configure
$ make
$ make install




### Install Python 2.5




#### Download Python 2.5 source package, unzip, and enter working directory:




> $ cd
$ wget http://python.org/ftp/python/2.5/Python-2.5.tgz
$ tar xvfz Python-2.5.tgz
$ cd Python-2.5




#### Configure Python 2.5 so that we're not overwriting the system's python installation:




> $ ./configure --prefix=/opt/python2.5 --with-threads --enable-shared




#### Make and install:




> $ make
$ make install




#### Add an alias to root's .bash_profile:




> alias python='/opt/python2.5/bin/python'




#### Make a symbolic link:




> $ ln -s /opt/python2.5/bin/python /usr/bin/python2.5




#### Configure ld to find your shared libs:




> $ cat >> /etc/ld.so.conf.d/opt-python2.5.conf
/opt/python2.5/lib (hit enter)
(hit ctrl-d to return to shell)
$ ldconfig




#### Test python 2.5 installation:




> $ python


You should get an interactive Python 2.5 session like:


> Python 2.5 (r25:51908, Nov  9 2008, 23:18:24)
[GCC 4.1.2 20071124 (Red Hat 4.1.2-42)] on linux2
Type "help", "copyright", "credits" or "license" for more information.

>>>

Hit ctrl-d to exit




#### Install setuptools:




> $ cd
$ wget http://pypi.python.org/packages/2.5/s/setuptools/setuptools-0.6c9-py2.5.egg
$ sh setuptools-0.6c9-py2.5.egg --prefix=/opt/python2.5




#### Install MySQLdb package:




> $ cd
$ wget http://internap.dl.sourceforge.net/sourceforge/mysql-python/MySQL-python-1.2.2.tar.gz
$ tar xvfz MySQL-python-1.2.2.tar.gz
$ cd MySQL-python-1.2.2
$ python setup.py build
$ python setup.py install




#### Verify Python 2.5 Installation:




> $ cd
$ python
Python 2.5 (r25:51908, Nov  9 2008, 23:18:24)
[GCC 4.1.2 20071124 (Red Hat 4.1.2-42)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import sqlite3
>>> import MySQLdb
>>>




### Installing mod_wsgi




#### What is mod_wsgi?


The aim of mod_wsgi is to implement a simple to use Apache module which can host any Python application which supports the Python WSGI interface. The module would be suitable for use in hosting high performance production web sites, as well as your average self managed personal sites running on web hosting services.


#### Why mod_wsgi?


The consensus seems to be that mod_wsgi is the prefered Apache module (as opposed to mod_python).  It's stable, less memory intensive, and faster.


#### Configure mod_wsgi to link with Python 2.5 shared libs:




> $ cd /opt/python2.5/lib/python2.5/config
$ ln -s ../../libpython2.5.so .
$ cd
$ wget http://modwsgi.googlecode.com/files/mod_wsgi-2.3.tar.gz
$ tar xvfz mod_wsgi-2.3.tar.gz
$ cd mod_wsgi-2.3
$ ./configure --with-python=/opt/python2.5/bin/python




#### Make and install:




> $ make
$ make install


It’s important to read the mod_wsgi docs on building with shared libs - mod_wsgi should compile to around 250kb.


#### Confirm that the size of mod_wsgi.so is around 250kb:




> $ ls -Al /usr/local/apache/modules/mod_wsgi.so




#### Add mod_wsgi to Apache 2.x:


With cPanel/WHM you can simply add these lines to your 'Pre-Virtualhost Include' file accessible in WHM:


> LoadModule wsgi_module /usr/local/apache/modules/mod_wsgi.so
AddHandler wsgi-script .wsgi




### Setup a Django 1.0 test project


Create a new user/domain via WHM/cPanel


#### Edit users .bash_profile:




> alias python='/opt/python2.5/bin/python'
export PYTHONPATH='$PYTHONPATH:/home/username/sites/domain.com'




#### Create a new project directory:




> $ mkdir -p /home/username/sites/domain.com
$ cd /home/username/sites/domain.com
$ svn co http://code.djangoproject.com/svn/django/trunk/ django-trunk
$ ln -s django-trunk/django django
$ mkdir .python-eggs
$ chmod 777 .python-eggs
$ /home/username/sites/domain.com/django/bin/django-admin.py startproject testproject
$ chown -R username: /home/username/sites




#### Create your apps .wsgi script:




> $ pico /home/username/public_html/test.wsgi

#!/opt/python2.5/bin/python
import os, sys
sys.path.insert(0,'/home/username/sites/domain.com')
os.environ['DJANGO_SETTINGS_MODULE'] = 'testproject.settings'
os.environ['PYTHON_EGG_CACHE'] = '/home/username/sites/domain.com/.python-eggs'
import django.core.handlers.wsgi
application = django.core.handlers.wsgi.WSGIHandler()

$ chown username: /home/username/public_html/test.wsgi




#### Setup a vhost include (as root)




> $ mkdir -p /usr/local/apache/conf/userdata/std/2/username/domain.com
$ pico /usr/local/apache/conf/userdata/std/2/username/domain.com/vhost.conf

<IfModule mod_alias.c>
Alias /robots.txt /home/username/sites/domain.com/testproject/media/robots.txt
Alias /site_media /home/username/sites/domain.com/testproject/media
Alias /admin_media /home/username/sites/domain.com/django/contrib/admin/media
</IfModule>

<IfModule mod_wsgi.c>
# See the link below for an introduction about this mod_wsgi config.
# http://groups.google.com/group/modwsgi/browse_thread/thread/60cb0ec3041ac1bc/2c547b701c4d74aa

WSGIScriptAlias / /home/username/public_html/test.wsgi
WSGIDaemonProcess username processes=7 threads=1 display-name=%{GROUP}
WSGIProcessGroup username
WSGIApplicationGroup %{GLOBAL} 
</IfModule>

# This fixes the broken ErrorDocument directive we inherit that breaks auth
# if we use a WSGI app.
ErrorDocument 401 "Authentication Error"
ErrorDocument 403 "Forbidden"

$ /scripts/verify_vhost_includes
$ /scripts/ensure_vhost_includes --user=username


domain.com should now be serving a django site.


#### To restart a django instance:




> $ touch ~/public_html/test.wsgi
