---
date: '2008-04-09 16:20:47'
layout: post
slug: svn-automatically-setting-properties
status: publish
title: SVN Automatically Setting Properties
wordpress_id: '39'
categories:
- Development
- Infrastructure
tags:
- auto-props
- enable-auto-props
- propset
- svn
---

It's quite the pain in the ass to manually propset for each new file added.




Edit the file .subversion/config and scroll down towards the bottom... find the **[miscellany] **block and uncomment the line **enable-auto-props = true**




In the group of options below titled **[auto-props] **and add a line like:




> * = svn:keywords=Id Date LastChangedBy Revision



