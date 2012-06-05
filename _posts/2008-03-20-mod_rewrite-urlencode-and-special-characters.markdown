---
date: '2008-03-20 14:08:32'
layout: post
slug: mod_rewrite-urlencode-and-special-characters
status: publish
title: mod_rewrite urlencode and special characters
wordpress_id: '37'
categories:
- Development
- Infrastructure
tags:
- mod_rewrite
- urlencode
---

If you're developing a website which uses mod_rewrite rules to redirect to a single point of entry (in which behind the scenes the actual url request gets passed as a query string parameter to "index.php" for example)... something like the following:
`
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php?url=$1 [L]
`

If you want to be able to hand-off parameters which contain special characters that would normally get urlencoded and urldecoded mod_rewrite will interfere.  The trick is to urlencode the portion of the request URL that needs to preserve special characters TWICE!
