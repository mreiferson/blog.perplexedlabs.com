---
date: '2009-02-08 23:15:58'
layout: post
slug: getting-started-with-django-and-python-first-impressions
status: publish
title: Getting Started with Django and Python - First Impressions
wordpress_id: '155'
categories:
- Development
- Django
tags:
- centos
- Django
- Python
---

I decided to spend the weekend getting more comfortable working and developing in Django (and coincidentally this MacBook Pro).  I've learned a lot already and I think this post might help some coming from other web development architectures to Django.

In a previous post I discussed [installing Django and getting it to play nice with Apache 2 on a CentOS 5/cPanel server](http://www.perplexedlabs.com/2008/11/10/setup-python-25-mod_wsgi-and-django-10-on-centos-5-cpanel/).  Updating to <del>Django 1.02</del> the latest Django codebase is easy if you followed my previous post, simply do an **svn up** in your django-trunk directory.

The first item that makes sense to tweak is Django's settings.  Editing your **settings.py** file in your project's root directory is fairly straightforward.  Fill in your database settings, adjust your time zone - all that good stuff.  Also, if you followed my previous post, you'll want to make a modification to your **ADMIN_MEDIA_PREFIX**:

```python
ADMIN_MEDIA_PREFIX = '/admin_media/'
```

Initially, you'll also want to use the (excellent) built in administrative interface.  Add it to your **INSTALLED_APPS** tuple at the bottom of **settings.py**:

```python
INSTALLED_APPS = (
    #...
    'django.contrib.admin'
)
```

To test your database connectivity and have Django setup tables required by the default apps that are placed in **INSTALLED_APPS** (including the admin app you just added) run the **manage.py syncdb** command:



> 
python manage.py syncdb




This brings up an issue I came across.  On my server I have a 2nd Python installation (v2.5 for Django development purposes).  You'd like to be able to give +x permissions to manage.py and simply do:



> 
./manage.py syncdb




Internally, the _shebang_ of Django's Python files is **#!/usr/bin/env python**.  The developers, knowing that a Python installation could be anywhere on a given system, obviously went the reliable route and utilize /usr/bin/env (which is essentially guaranteed to exist in that location) to search for a Python interpreter in the system's path.  However, on a system where your desired Python installation won't be found ahead of the system's default (perhaps, required) installation, running the script directly will cause it to be executed by the wrong Python interpreter.

The composition of a Django web application (site) is: _a site is composed of projects which are composed of apps_.  Apps can be completely decoupled and can be re-used on other projects (and therefor sites).  Apps seem to be the functional equivalent of Rails plugins.

Creating your first app is as simple as executing the **python manage.py startapp appname** command and adding it to your list of **INSTALLED_APPS**.  Personally, the fact that you have to add _anything_ to a list seems cumbersome.  Is it not possible to have them placed in a directory by convention that would automatically load them?  Perhaps this is due to the fact that you may want to take advantage of certain Django bundled apps, such as auth, admin, etc.  In which case it wouldn't make sense to copy them into an "apps" directory under your project root.

After you've created your first app you then create your models.  If you have an familiarity with an **O**bject **R**elation **M**apper (ORM) Django's won't come as any surprise.  Your models extend Django's built in **models.Model** class and define both the database fields as properties and the actions as methods.

As the official Django tutorial suggests, it's helpful to overwrite some base model methods with your own custom methods to implement some important functionality.  In particular **__unicode__()**.  Internally **__str__()** calls **__unicode__()**, so whereas in Python your classes would implement **__str__()**, in Django you implement **__unicode__()**.  This provides a human readable representation of your object when dealing with the built-in Django admin interface, or the Django shell.

_Edit below: It was late when I wrote this, sorry!_
<del>It's also worth noting that it was somewhat surprising to me that certain files were created automatically for you with the **manage.py startapp** command while others weren't (such as models.py).  This certainly isn't something awful, just seemed a bit inconsistent.</del>  This extends to setting up templates.  Again, it's a manual process.  You first create the templates directory and then specify it in your project's **settings.py** file.  Couldn't some of this have been automated?  If you want to modify Django's built-in admin interface template you have to copy the desired file into your templates directory, manually.  Perhaps this just takes some getting used to.  Fair enough.

One issue the official Django tutorial didn't touch upon is setting up your projects "home" view (accessing / ).  It's simple, your _project_ can define it's own views that can be added to your **urls.py** file.

```python
urlpatterns = patterns('',
    (r'^$', 'myproject.views.index'),
    (r'^admin/(.*)', admin.site.root),
)
```

And in your _project's_ **views.py** file:

```python
from django.http import HttpResponse

def index(request):
        return HttpResponse('Hello, World!')
```

It also occurred to me that if apps are meant to be decoupled and packageable, and there is only one template directory (all apps template files go in this directory).  What IS the process for packaging and deploying an app into another project?  MORE manual copying?

SHRUG

That's all for now, I'm sure I'll have plenty more to comment about as I experiment.  I do like what I see though and I'm sure I have TONS to learn.  Also - please, correct me if I'm mistaken about anything above.
