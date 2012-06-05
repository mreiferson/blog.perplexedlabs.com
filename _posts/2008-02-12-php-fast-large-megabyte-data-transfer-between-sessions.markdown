---
date: '2008-02-12 12:24:04'
layout: post
slug: php-fast-large-megabyte-data-transfer-between-sessions
status: publish
title: PHP fast, large (megabyte), data transfer between sessions
wordpress_id: '29'
categories:
- Development
- PHP
tags:
- php
- serialize
- session
- unserialize
- var_export
---

The ability to transfer data from one executing script to the next as components of a web application is of the utmost importance.  Sometimes the objects in transit can grow in size to 10's of megabytes of data.  Nested classes, arrays of smaller classes, arrays of data, indexes, etc.

I've been on a pretty long optimization kick for a lot of the production applications that are used on a daily basis.  Most applications use an "extended database listing" class that extracts raw data from a mySQL table and provides interfaces to add columns using that base data and sort, format, and highlight the data.  The process works in two phases - the second being an AJAX request to actually deliver the HTML output to the browser.  In choosing to go this route (the originating page now finishes execution faster and displays faster) it's necessary to pass the list object from the originating script to the AJAX display helper script.  It is this object that, depending on the size and data in the list, can become enormous.

Initially I had been using the tried and true serialize() / unserialize() on the object and storing it in the $_SESSION superglobal.  As soon as the object becomes anywhere "unusually" large, the unserialize call will take an exponentially increasing # of seconds to complete.  Not satisfied with this situation I searched and searched for a solution.

One solution I've come across that cuts execution time by 50% is the use of var_export() instead.  var_export() will, by default, output a php parsable representation of the variable passed to it.  In my case, the object being exported was a class.  PHP > 5.1.0 changed the way var_export worked on class objects.  You now have to implement the magic function __set_state() in your class in order to correctly handle a class exported with var_export().  On the other end of the process, where you would normally use unserialize(), now all you do is include the file you exported to.

Here is my implementation of the __set_state() magic function:
```php
public static function __set_state($array)
{
    $obj = new cListExtDB;
    foreach($array as $field => $val) {
        $obj->$field = $array[$field];
    }
    return $obj;
}
```

The following example illustrates the exporting side of this procedure:
```php
');
?>
```

And in the AJAX helper script, the following example illustrates the importing side of this procedure:
```php

``` 
