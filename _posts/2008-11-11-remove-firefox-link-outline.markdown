---
date: '2008-11-11 13:15:06'
layout: post
slug: remove-firefox-link-outline
status: publish
title: Remove Firefox Link Outline
wordpress_id: '62'
categories:
- CSS
- Development
tags:
- CSS
- firefox
---

Add these rules to your stylesheet to remove the dotted border that firefox places around (active) links.

```css
/* remove firefox link outline */
a { outline: none; }
:-moz-any-link:focus { outline: none; }
``` 
