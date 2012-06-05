---
date: '2010-02-08 08:00:32'
layout: post
slug: deployment-using-capistrano-and-webistrano-via-rails-and-phusion-passenger
status: publish
title: Deployment Using Capistrano / Webistrano via Rails / Phusion Passenger
wordpress_id: '408'
categories:
- Development
- Django
- Infrastructure
- PHP
- Python
- Ruby
- Ruby on Rails
tags:
- deployment
- Django
- mod_wsgi
- mongrel
- passenger
- php
- phusion passenger
- Python
- rails
- ruby
- subversion
---

I finally got around to setting up a more sophisticated deployment system for some of my apps.  These apps include some built on a custom PHP framework and others that are Python / Django apps.  I figured I'd share my experience...

Why is a high-level deployment infrastructure important?  Deployment is something that should be simple, accessible, and repeatable.  It should be as close to a "single click" as possible.  Previously, for me, it was a bash script that exported some SVN branches.  While this worked fine, as projects progress, you want some accountability, history, and the ability to roll back mission critical applications when something goes wrong with a deploy.

[Capistrano](http://www.capify.org/) is an open source, command line, deployment tool that provides all of these features.  It's written in Ruby.  You leverage a variety of built in "recipes" (Capistrano's term for a deployment script) that execute certain procedures to deploy an app.  Out-of-the-box it's ideally built to deploy a Rails app.  However, after some minor tweaks it can deploy most anything and do it well.  It can restart servers, update symlinks, change permissions - pretty much anything.  It assumes you access your POSIX compliant server via SSH via the same password (or have ssh keys setup).

[Webistrano](http://labs.peritor.com/webistrano) is an open source web front-end for Capistrano.  It's a convenience layer that abstracts the command line away and provides an interface to perform the same tasks.  This interface shows history as well as providing a convenient GUI for creating new deployment projects, stages, and recipes.  Highly recommended.

Let's get down to business.  This post makes a few assumptions about things you've already installed and used previously.





  * [Ruby 1.8.5](http://www.ruby-lang.org/en/)+

  * [RubyGems](http://rubyforge.org/projects/rubygems/)

  * [MySQL 5.0](http://www.mysql.com/)+

  * [Apache 2](http://www.apache.org/)+

  * [Subversion](http://subversion.tigris.org/) and a repository containing the "production" branch of your app.




### Installing Capistrano



Well, this is an easy one (you probably want to do this as root):



> 
gem install capistrano






### Installing Webistrano



Also fairly easy, with a little splash of configuration.



> 
# wget http://labs.peritor.com/webistrano/attachment/wiki/Download/webistrano-1.4.zip
# unzip webistrano-1.4.zip
# mv webistrano-1.4 /path/to/where/you/want/webistrano




Setup the database tables and create a new webistrano user (obviously be conscious of your security preferences for access to your database in the host and password portions):



> 
# mysql
mysql> CREATE DATABASE `webistrano`;
mysql> CREATE USER 'webistrano'@'localhost' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON `webistrano`.* TO 'webistrano'@'localhost' WITH GRANT OPTION;




Now, in the directory where you placed webistrano you're going to want to copy _config/database.yml.sample_ to _config/database.yml_.  Edit this file, in the production area, to match your database settings.  By default the file expects a socket to connect, you can chase this by specifying _host:_ and _port:_.  (Keep in mind Webistrano is simply a Rails app).

You should now be able to have Rails migrate the new database you created.  In the webistrano directory:



> 
# RAILS_ENV=production rake db:migrate




Finally, copy _config/webistrano_config.rb.sample_ to _config/webistrano_config.rb_ and edit according to your preferred mail settings.

We can now test to see if webistrano is working properly by serving it via mongrel:



> 
# ruby script/server -d -e production -p 3000




This starts a single mongrel daemon, using the production environment, listening on port 3000.  You should now be able to hit http://127.0.0.1:3000/ and get the Webistrano login prompt.  If this is working, kill that mongrel instance.

For longer term serving I decided to go with Phusion Passenger (essentially mod_rails for Apache).  It's a nearly zero configuration solution for serving a rails app and will feel at home to anyone with experience serving PHP apps via Apache and mod_php.



### Installing Phusion Passenger



Again, as root:



> 
# gem install passenger
# passenger-install-apache2-module




The second command will invoke an installer which compiled Passenger and provides instructions on integrating it into your Apache config.  Essentially, edit your httpd.conf as follows (**these were specific to my install, make sure to use the ones provide by the installer for you**):



> 
LoadModule passenger_module /usr/lib/ruby/gems/1.8/gems/passenger-2.2.9/ext/apache2/mod_passenger.so
PassengerRoot /usr/lib/ruby/gems/1.8/gems/passenger-2.2.9
PassengerRuby /usr/bin/ruby




Now you can simply add VirtualHost entries to your httpd.conf for any of your Rails apps.  Let's add one for Webistrano:



> 
<VirtualHost *:80>
ServerName webistrano.mydomain.com
DocumentRoot /path/to/webistrano/public
</VirtualHost>




Yes, Passenger makes it that simple.  Add configuration directives as needed for your environment.

Now Webistrano should be serving from the VirtualHost you specified, seamlessly, via Passenger.



### Deploying A Non-Rails App



Now the fun stuff.

Capistrano breaks things down into projects, stages, and recipes.  Each app you want managed by capistrano should be it's own project.  Each project should have a stage for at least production and optionally staging and development.

Hosts are added globally and form the targets of a deploy for any given project.  Hosts can include web, app, and database servers.

Deployments in Capistrano are done to a child directory under "releases" named via the date and time of the deployment.  By default 5 releases are kept and available to rollback to.  Upon successful deployment a symlink (default is called "current" and can be modified via the _current_path_ configuration variable) is updated to that release directory.  It is this symlink that should be targeted by your webserver (your DocumentRoot in Apache).

Capistrano also creates a "shared" directory that is symlinked to in each release useful for storing logs and other data that should be maintained through each deployment.

For non-rails apps you'll use the "Pure File" project type when creating your new project.  Upon project creation you can add configuration variables specific to your project.  I recommend using _:export_ instead of _:checkout_ for _deploy_via_ for production subversion deployments as this doesn't expose .svn directories.  Use an SSH user that has enough permissions to create directories where your deploy will occur or, specify _use_sudo_ to true and create a new configuration variable _admin_runner_ and set it to the same user as _runner_.

Add a stage to your new project for "production".  In the "Manage Hosts" page add a new host for each of your application servers.  Then add each host as a target of your "production" stage of your project.

At this point you should be able to execute the "Setup" task for your "production" stage.  This is a one time task that simply creates the directories.

Assuming this went successfully, try doing a "Deploy" and see if that finishes without error.  You might have to play around with permissions and other minor issues - post a comment if you have any specific questions.

For my PHP framework there are a couple specific tasks I wanted to run in addition to the default Capistrano tasks.  You do this by creating custom recipes in the "Manage Recipes" page in Webistrano.  Recipes are simply procedures written in ruby.  Here's what my recipe looks like:

```ruby
namespace :deploy do
	task :setup, :except => { :no_release => true } do
		dirs = [deploy_to, releases_path, shared_path]
		dirs += shared_children.map { |d| File.join(shared_path, d) }
		run "#{try_sudo} mkdir -p #{dirs.join(' ')} && #{try_sudo} chmod g+w #{dirs.join(' ')}"
		run "chmod 777 #{shared_path}/log"
	end

	task :finalize_update, :except => { :no_release => true } do
		run "mkdir -p #{latest_release}/app/tmp"
		run "chmod -R 777 #{latest_release}/app/tmp"
		run "rm -rf #{latest_release}/app/logs"
		run "ln -s #{shared_path}/log #{latest_release}/app/logs"
		run "cp #{latest_release}/public_html/.htaccess-production #{latest_release}/public_html/.htaccess"
		run "cp #{latest_release}/app/config/config-production.php #{latest_release}/app/config/config.php"
		run "cp #{latest_release}/app/config/db-default.php #{latest_release}/app/config/db.php"
		run "cp #{latest_release}/app/config/memcache-default.php #{latest_release}/app/config/memcache.php"
	end
end
```

If you're not familiar with Ruby - what this code is essentially doing is overwriting two tasks in the :deploy namespace with my custom code.

The first, :setup, simply duplicates the base :setup functionality discussed above (creating the releases and shared directories) and chmods the shared log directory to be writable.

The second, :finalize_update, performs a variety of configuration tasks for a PHP app built with my framework.  Also, you'll notice that I'm removing my app's logs directory and symlinking to the shared log directory.  This way all releases will log to the same directory, consistently.  

In my case all of these procedures are command line instructions.  Alternatively, you can do a variety of things leveraging the full breadth of the Ruby language and any gem you'd like to introduce.   Things such as accessing your CDN API to clear image, JS, or CSS caching, etc.



### Deploying Django Apps



First off it's worth noting that I serve my Django apps via mod_wsgi.  To make the deployment process easier here's what my app.wsgi script looks like:

```python
import os
import sys

appdir = os.path.normpath(os.path.join(os.path.realpath(os.path.dirname(__file__)), '..'))
sys.path.insert(0, appdir)
os.environ['DJANGO_SETTINGS_MODULE'] = 'settings'
os.environ['PYTHON_EGG_CACHE'] = os.path.join(appdir, '.python-eggs')
import django.core.handlers.wsgi
application = django.core.handlers.wsgi.WSGIHandler()
```

This code allows us to avoid having to hardcode paths in the wsgi script (and thus avoid having to change them when we deploy).  It assumes the following directory structure:



> 
.python-eggs (egg cache)
apps (apps path is added to python system path in settings.py)
public (where your .wsgi script resides)
site_media
templates
settings.py
settings-production.py (used for deploy)
urls.py
...




If you follow this convention, the following Capistrano recipe works great:

```ruby
namespace :deploy do
	task :setup, :except => { :no_release => true } do
		dirs = [deploy_to, releases_path, shared_path]
		dirs += shared_children.map { |d| File.join(shared_path, d) }
		run "#{try_sudo} mkdir -p #{dirs.join(' ')} && #{try_sudo} chmod g+w #{dirs.join(' ')}"
		run "chmod 777 #{shared_path}/log"
	end

	task :finalize_update, :except => { :no_release => true } do
		run "rm -rf #{latest_release}/logs"
		run "ln -s #{shared_path}/log #{latest_release}/logs"
		run "cp #{latest_release}/settings-production.py #{latest_release}/settings.py"
		run "mkdir -p #{latest_release}/.python-eggs"
		run "chmod 777 #{latest_release}/.python-eggs"
	end
end
```



### Fin



This should give you a nice intro to leveraging Capistrano via Webistrano.  Feel free to comment with questions, suggestions, or anything else!
