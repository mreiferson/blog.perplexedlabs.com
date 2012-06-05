---
date: '2009-08-13 09:00:38'
layout: post
slug: adventures-in-django-and-python-part-iii
status: publish
title: Adventures in Django and Python - Part III
wordpress_id: '307'
categories:
- Development
- Django
- Python
tags:
- Django
- Python
- session
- ternary
---

_Read my previous two posts on Django and Python - [Part I](http://blog.perplexedlabs.com/2009/02/08/getting-started-with-django-and-python-first-impressions/) and [Part II](http://blog.perplexedlabs.com/2009/03/20/django-and-python-first-impressions-part-ii/)_

I've been working on a project management tool suite in Django.  It's been a great side project to really experiment with Django in real-world scenarios.



### Forms



At times I feel like I fight with newforms.  In particular, it lacks the ability to specify basic class or style attributes for a given form field from within the template.  I'd like to be able to more finely tune the display of the field, directly within the template, with a style attribute or a class.  Is it suggested you write your own custom form field widget for a single element?  I've been getting around this by doing the following:

```python
<input type="text" name="{{ todo_form.item.name }}" style="width: 720px;"/>
```

This gets more complicated if you want to set a style attribute for a form field that's a select box (for a ForeignKey model field, for example).  

```python
<label for="category">Category:</label> <select name="{{ category_form.project.name }}" style="width: 221px;">
{% for choice_val, choice_label in category_form.project.field.choices %}
	<option value="{{ choice_val }}">{{ choice_label }}</option>
{% endfor %}
```

Is this a good use case for template tags?  I feel like I'm missing something here.

On the positive side, it was an absolute pleasure to work with multiple forms on a single page submitted to and processed by a single view.  This is primarily thanks to prefixes.  Excellent, that's how easy it should be.



### Ternary Operator



**Update:** _It's been pointed out in comments (thanks!) that Python 2.5 introduced a ternary operator.  It's syntax is as follows:_

```python
label = "true" if booleanVariable else "false"
```

I also ran into a minor Python syntax issue.  I love the ternary operator in languages that offer it.  It's a concise, one-line, syntax for an if-else clause.  Consider the following PHP code:

```php
$label = $booleanVariable ? 'true' : 'false';

// the above is identical to the following block:
if($booleanVariable) {
   $label = 'true';
} else {
   $label = 'false';
}
```

Python unfortunately lacks this syntactic sugar.  Fortunately, however, you can effectively accomplish the same thing by doing this:

```python
label = (booleanVariable and 'true' or 'false')

# the above is equivalent to the following block:
if booleanVariable:
   label = 'true'
else:
   label = 'false'
```



### Sessions



Django has built in support for sessions.  By default, sessions last longer than the lifecycle of the user's browser.  I personally think it should be the other way around.  It's easily changed though (in your settings.py):

```python
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```



### Views



In one of my views I wanted to test whether a filtered result set was empty or not.  I was curious whether this was the "pythonic" way to accomplish this:

```python
account = get_object_or_404(Account, pk=account_id)
if account.useraccount_set.filter(user__exact=request.user) != []: 
```

Also, with respect to views and passing context to the response, sometimes it's an efficient shortcut to use **locals()** instead of explicitly typing out all the variables you'd like to expose.  locals() returns a dictionary of all variables defined within the local scope.

```python
def myview(request, id):
    account = get_object_or_404(Account, pk=id)
    new_account_form = NewAccountForm()

    return render_to_response('myview.html', locals())
```

More soon!
