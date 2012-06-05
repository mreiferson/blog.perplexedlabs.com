---
date: '2010-03-02 08:00:42'
layout: post
slug: django-up-in-your-cron
status: publish
title: Django Up In Your CRON
wordpress_id: '354'
categories:
- Development
- Django
- Python
tags:
- cron
- Django
- Python
---

For one off scripts for a particular project:

```python
#!/usr/bin/env python

from django.core.management import setup_environ
from myapp import settings
setup_environ(settings)

# do some stuff
``` 
