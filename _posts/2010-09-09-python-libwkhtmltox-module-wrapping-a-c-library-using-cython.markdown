---
date: '2010-09-09 07:00:50'
layout: post
slug: python-libwkhtmltox-module-wrapping-a-c-library-using-cython
status: publish
title: Python libwkhtmltox module - wrapping a C library using Cython - convert HTML
  to PDF
wordpress_id: '483'
categories:
- Development
- Django
- Python
- Random
tags:
- binding
- c++
- cython
- libwkhtmltox
- module
- Python
- wkhtmltoimage
- wkhtmltopdf
- wrapper
---

First of all, big shout out to antialize for creating [wkhtmltopdf](http://code.google.com/p/wkhtmltopdf/) ([github repo](http://github.com/antialize/wkhtmltopdf)).

Also, this project is being hosted on GitHub @ [http://github.com/mreiferson/py-wkhtmltox](http://github.com/mreiferson/py-wkhtmltox).



## [wkhtmltox](http://code.google.com/p/wkhtmltopdf/)



What is wkhtmltox you ask?  It's a utility built on [Nokia's Qt](http://qt.nokia.com/) framework for converting HTML (including images, CSS, **and Javascript**) to a PDF or image.  When Qt introduced it's [webkit module](http://doc.trolltech.com/4.6/qtwebkit.html) it made it relatively easy to leverage it's rendering engine to produce high quality output.

wkhtmltox 0.10.0beta5 introduced libwkhtmltox - a simple C API making it possible to embed this functionality in higher-level scripting languages.  I jumped at the opportunity to build Python bindings...

First I tried [SIP](http://www.riverbankcomputing.co.uk/software/sip/intro), which is actually used for the Qt bindings.  A variety of issues coupled with poor documentation led me to search for other solutions.  I won't bore you with the details because it seems there are better ways...



## [Cython](http://www.cython.org/)



Note: I'm not talking about CPython (the default Python implementation written in C).  I'm talking about a toolset to build C extensions for Python.  The two major use cases being speed or, in my case, wrapping a C library.

It's really easy to get up and running, Cython the language is a hybrid of C and Python.  You write scripts with a .pyx extension, make things modular with .pxd files, and handle all the building and installation with [distutils](http://docs.python.org/distutils/).

Let's take a look at the API libwkhtmltox exposes (I'm only showing lines relevant to this post):

```c
struct wkhtmltopdf_global_settings;
typedef struct wkhtmltopdf_global_settings wkhtmltopdf_global_settings;

struct wkhtmltopdf_object_settings;
typedef struct wkhtmltopdf_object_settings wkhtmltopdf_object_settings;

struct wkhtmltopdf_converter;
typedef struct wkhtmltopdf_converter wkhtmltopdf_converter;

CAPI int wkhtmltopdf_init(int use_graphics);
CAPI int wkhtmltopdf_deinit();

CAPI const char * wkhtmltopdf_version();

CAPI wkhtmltopdf_global_settings * wkhtmltopdf_create_global_settings();
CAPI wkhtmltopdf_object_settings * wkhtmltopdf_create_object_settings();

CAPI int wkhtmltopdf_set_global_setting(wkhtmltopdf_global_settings * settings, const char * name, const char * value);
CAPI int wkhtmltopdf_set_object_setting(wkhtmltopdf_object_settings * settings, const char * name, const char * value);

CAPI wkhtmltopdf_converter * wkhtmltopdf_create_converter(wkhtmltopdf_global_settings * settings);
CAPI void wkhtmltopdf_destroy_converter(wkhtmltopdf_converter * converter);

CAPI int wkhtmltopdf_convert(wkhtmltopdf_converter * converter);
CAPI void wkhtmltopdf_add_object(wkhtmltopdf_converter * converter, wkhtmltopdf_object_settings * setting, const char * data);

CAPI int wkhtmltopdf_http_error_code(wkhtmltopdf_converter * converter);
```

And let's also look at the basic example provided with the wkhtmltopdf source distribution (again, only relevant lines shown):

```c
wkhtmltopdf_init(false);
gs = wkhtmltopdf_create_global_settings();
wkhtmltopdf_set_global_setting(gs, "out", "test.pdf");
os = wkhtmltopdf_create_object_settings();
wkhtmltopdf_set_object_setting(os, "page", "http://doc.trolltech.com/4.6/qstring.html");
c = wkhtmltopdf_create_converter(gs);
wkhtmltopdf_add_object(c, os, NULL);
wkhtmltopdf_convert(c);
wkhtmltopdf_destroy_converter(c);
wkhtmltopdf_deinit();
```

I found it helpful to try to identify which methods needed to be exposed to user-space and which ones could be safely abstracted behind a cleanly wrapped Pythonic API.  I wrote the following test script to work towards:

```python
import wkhtmltox

pdf = wkhtmltox.Pdf()
pdf.set_global_setting('out', 'test.pdf')
pdf.set_object_setting('path', 'http://www.google.com')
pdf.convert()
```

The initialization of a Pdf instance handles all of the internal libwkhtmltox initialization.  See below for the .pyx script.  Of interest is how C structs and functions are exposed to the Cython script and then wrapped, with appropriate names, as Python methods of a class.  Astute observers will also notice that some of the function declarations differ slightly from the original libwkhtmltox header file.  In most cases Cython doesn't need the const, CAPI and other specific declarations.  Also, **bint** hints that even though the return value is an int we should convert this to a boolean on the Python side.

```python
cdef extern from "wkhtmltox/pdf.h":
    struct wkhtmltopdf_converter:
        pass
    
    struct wkhtmltopdf_object_settings:
        pass
    
    struct wkhtmltopdf_global_settings:
        pass
    
    bint wkhtmltopdf_init(int use_graphics)
    bint wkhtmltopdf_deinit()
    char *wkhtmltopdf_version()
    
    wkhtmltopdf_global_settings *wkhtmltopdf_create_global_settings()
    wkhtmltopdf_object_settings *wkhtmltopdf_create_object_settings()
    
    bint wkhtmltopdf_set_global_setting(wkhtmltopdf_global_settings *settings, char *name, char *value)
    bint wkhtmltopdf_get_global_setting(wkhtmltopdf_global_settings *settings, char *name, char *value, int vs)
    bint wkhtmltopdf_set_object_setting(wkhtmltopdf_object_settings *settings, char *name, char *value)
    bint wkhtmltopdf_get_object_setting(wkhtmltopdf_object_settings *settings, char *name, char *value, int vs)
    
    wkhtmltopdf_converter *wkhtmltopdf_create_converter(wkhtmltopdf_global_settings *settings)
    void wkhtmltopdf_destroy_converter(wkhtmltopdf_converter *converter)
    
    bint wkhtmltopdf_convert(wkhtmltopdf_converter *converter)
    void wkhtmltopdf_add_object(wkhtmltopdf_converter *converter, wkhtmltopdf_object_settings *setting, char *data)
    
    int wkhtmltopdf_http_error_code(wkhtmltopdf_converter *converter)


cdef extern from "wkhtmltox/image.h":
    struct wkhtmltoimage_global_settings:
        pass
    
    struct wkhtmltoimage_converter:
        pass
    
    bint wkhtmltoimage_init(int use_graphics)
    bint wkhtmltoimage_deinit()
    char *wkhtmltoimage_version()
    
    wkhtmltoimage_global_settings *wkhtmltoimage_create_global_settings()
    
    bint wkhtmltoimage_set_global_setting(wkhtmltoimage_global_settings *settings, char *name, char *value)
    bint wkhtmltoimage_get_global_setting(wkhtmltoimage_global_settings *settings, char *name, char *value, int vs)
    
    wkhtmltoimage_converter *wkhtmltoimage_create_converter(wkhtmltoimage_global_settings *settings, char *data)
    void wkhtmltoimage_destroy_converter(wkhtmltoimage_converter *converter)
    
    bint wkhtmltoimage_convert(wkhtmltoimage_converter *converter)
    
    int wkhtmltoimage_http_error_code(wkhtmltoimage_converter *converter)


cdef class Pdf:
    cdef wkhtmltopdf_global_settings *_c_global_settings
    cdef wkhtmltopdf_object_settings *_c_object_settings
    cdef bint last_http_error_code
    
    def __cinit__(self):
        wkhtmltopdf_init(0)
        self._c_global_settings = wkhtmltopdf_create_global_settings()
        self._c_object_settings = wkhtmltopdf_create_object_settings()
    
    def __dealloc__(self):
        wkhtmltopdf_deinit();
    
    def set_global_setting(self, char *name, char *value):
        return wkhtmltopdf_set_global_setting(self._c_global_settings, name, value)
    
    def set_object_setting(self, char *name, char *value):
        return wkhtmltopdf_set_object_setting(self._c_object_settings, name, value)
    
    def convert(self):
        cdef wkhtmltopdf_converter *c
        c = wkhtmltopdf_create_converter(self._c_global_settings)
        wkhtmltopdf_add_object(c, self._c_object_settings, NULL)
        ret = wkhtmltopdf_convert(c)
        self.last_http_error_code = wkhtmltopdf_http_error_code(c)
        wkhtmltopdf_destroy_converter(c)
        return ret
    
    def http_error_code(self):
        return self.last_http_error_code


cdef class Image:
    cdef wkhtmltoimage_global_settings *_c_global_settings
    cdef bint last_http_error_code
    
    def __cinit__(self):
        wkhtmltoimage_init(0)
        self._c_global_settings = wkhtmltoimage_create_global_settings()
    
    def __dealloc__(self):
        wkhtmltoimage_deinit();
    
    def set_global_setting(self, char *name, char *value):
        return wkhtmltoimage_set_global_setting(self._c_global_settings, name, value)
    
    def convert(self):
        cdef wkhtmltoimage_converter *c
        c = wkhtmltoimage_create_converter(self._c_global_settings, NULL)
        ret = wkhtmltoimage_convert(c)
        self.last_http_error_code = wkhtmltoimage_http_error_code(c)
        wkhtmltoimage_destroy_converter(c)
        return ret
    
    def http_error_code(self):
        return self.last_http_error_code
```

This is really my first attempt to get something working.  I'm sure there are bugs and perhaps better ways to go about this.  I always welcome questions/feedback.

I'm going to continue to support this project at [http://github.com/mreiferson/py-wkhtmltox](http://github.com/mreiferson/py-wkhtmltox) - watch it!
