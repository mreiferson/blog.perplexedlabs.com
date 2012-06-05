---
date: '2010-09-15 08:49:41'
layout: post
slug: convert-html-to-pdf-in-php-libwkhtmltox-extension
status: publish
title: Convert HTML to PDF in PHP (libwkhtmltox extension)
wordpress_id: '502'
categories:
- Development
- PHP
tags:
- convert
- extension
- html to pdf
- libwkhtmltox
- pdf
- php
- wkhtmltopdf
---

A common problem when developing a web application is having producing a high-quality PDF out of an existing layout/view/template.  Perhaps for a reporting engine, an invoice, a receipt, or any number of other situations.

Often this involves using somewhat cryptic output primitives and creating the PDF by hand.  Wouldn't it be nice if there were a way to **re-use** all that beautiful HTML, CSS, and maybe even _Javascript_ that you already wrote?

Well, there is.  It's called [wkhtmltopdf](http://code.google.com/p/wkhtmltopdf/).  Normally a command line utility, with the release of 0.10.0_beta5 antialize included a simple C API to be able to build bindings in other popular languages.

I'm proud to announce the release of a PHP extension that facilitates the process of doing the conversion directly in PHP:

```php
<?php

wkhtmltox_convert('pdf',
    array('out' => 'test.pdf', 'imageQuality' => '95'), // global settings
    array(
        array('page' => 'http://www.visionaryrenesis.com/'),
        array('page' => 'http://www.google.com/', 'web.printMediaType' => true)
        ));

?>
```

I'm hosting the code at GitHub:  [http://github.com/mreiferson/php-wkhtmltox](http://github.com/mreiferson/php-wkhtmltox)

It was certainly interesting working with PHP under the hood but overall the process was pretty straightforward.  Keep in mind the function signatures may change a bit as the API matures.  Feedback welcome!
