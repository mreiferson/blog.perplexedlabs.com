---
date: '2009-03-20 10:05:51'
layout: post
slug: django-and-python-first-impressions-part-ii
status: publish
title: Django and Python First Impressions - Part II
wordpress_id: '190'
categories:
- Development
- Django
- Python
- Ruby on Rails
tags:
- Django
- Python
- rails
- ruby
---

I've spent more time with Django the past couple days.  Read my [installation guide](http://www.perplexedlabs.com/2008/11/10/setup-python-25-mod_wsgi-and-django-10-on-centos-5-cpanel/) and my [first impressions](http://www.perplexedlabs.com/2009/02/08/getting-started-with-django-and-python-first-impressions/) to get caught up.  I wanted to address a couple issues I came across as I was exposed to certain architectural designs of Django.

It might be helpful to note which books available today cover Django 1.0.  I do realize that the "official" Django book covering 1.0 is in the works, but, in the meantime I recommend [Python Web Development with Django (Developer's Library)](http://www.amazon.com/gp/product/0132356139?ie=UTF8&tag=perplabs-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0132356139)![](http://www.assoc-amazon.com/e/ir?t=perplabs-20&l=as2&o=1&a=0132356139) and [Pro Django (Expert's Voice in Web Development)](http://www.amazon.com/gp/product/1430210478?ie=UTF8&tag=perplabs-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=1430210478)![](http://www.assoc-amazon.com/e/ir?t=perplabs-20&l=as2&o=1&a=1430210478).

Lets start with the template system.  While Django's template system is powerful, I'm sure, it's basically a free for all of file names and directory structures.  You could conceivably create a single 'templates' directory and place all of a projects template files in that directory (naming the files as you please) for any and all the apps that compose the project.  While it's not recommended you actually do that, Django would support it because it doesn't seem to really dictate or enforce any particular convention.  I feel like Rails, comparatively speaking, provides solid conventions for a developer to follow in terms of file naming, directory naming, and directory structure.

I was also surprised at the fact that Django does NOT bundle a pluralization library.  Even simple cases (ending in **y** to **ies**) aren't handled automatically.  A model named 'Category' appears as 'Categorys'.  It does provide support for explicitly specifying plural names for Models via the **verbose_name_plural** Meta property:

```python
class Category(models.Model):
    name = models.CharField(max_length=128)
    class Meta:
        verbose_name_plural = 'categories'
```

Coming from a world of a custom built PHP framework - it's REALLY nice to have features like the command line sandbox to be able to play with Models and interact with the project's codebase.  It's also great that Django bundles a development webserver for a quick and easy create-test-edit cycle.  It recognizes changes to your code while it's running - there's no need to restart manually.

The effects are immeasurable with respect to the fact that Python functions are first-class.  Django uses this extensively (such as your URL configuration or default values for Models).  It's extremely intuitive to be able to do the following in your Model definition:

```python
class Post(models.model)
    stamp = models.DateTimeField(default=datetime.now)
```

Notice this wasn't written as **datetime.now()**.  That would actually execute the function when the class is declared and all entries would receive an identical stamp.  Instead we're passing a _reference_ to the function.  Django detects this and calls the function when the Model is instantiated.

I'm also excited about the way the Django framework handles requests and responses.  The concept of the view receiving an HttpRequest object, giving context, and returning an HttpResponse object just makes sense.  It's very much in line with the way HTTP works.  It's simple, powerful, and elegant.

I've read about ([and watched video of](http://www.youtube.com/watch?v=i6Fr65PFqfk)) some issues that people have with Django.  These usually refer to difficulties one _may_ have scaling it.  Lack of built-in support for multiple databases and sharding are cited, among other reasons.  I think, for what it's able to do right out of the box, it's fantastic.  Scaling (in general) isn't always straightforward.  In many cases it requires specific tools and solutions for the task at hand.  These issues should in no way prevent you from using Django for a project!

More soon.
