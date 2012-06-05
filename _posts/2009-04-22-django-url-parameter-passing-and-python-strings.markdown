---
date: '2009-04-22 11:18:01'
layout: post
slug: django-url-parameter-passing-and-python-strings
status: publish
title: Django URL Parameter Passing and Python Strings
wordpress_id: '224'
categories:
- Development
- Django
- Python
tags:
- Django
- Python
- string
---

This post is simply stating the obvious.  Sometimes even obvious things, in the wee hours of the morning, aren't so.

When you specify parameters in your URLconf like:

```python
urlpatterns = patterns('',
    url(r'^mark/(?P\d+)/(?P\d+)/$', views.mark, name='mark'),
)
```

Keep in mind that each captured argument is a Python string.  Even if the regex only captures integers - 'complete' is still passed as a Python string to your 'mark' view.

So if you intended to pass a 0 for false and 1 for true you must make sure to convert to an integer because only a string of length 0 is False.  ('0' == True)

```python
def mark(request, id, complete):
    todo = get_object_or_404(pk=id)
    todo.complete = int(complete)
    todo.save()
    return HttpResponse()
``` 
