---
date: '2010-03-15 08:00:07'
layout: post
slug: learning-vs-solving
status: publish
title: Learning vs. Solving
wordpress_id: '443'
categories:
- Development
- Random
tags:
- Random
- thoughts
---

Often times you're tasked with solving a problem you haven't faced before, requiring the use of technologies you haven't previously been exposed to.  This is a great thing!  These experiences are the stuff of legend - continuing deep into the night as your curiosity peaks.

When delivery of the solution makes a difference to somebody's bottom line you have to balance the opportunity as a means to learn with your desire to deliver for a customer.

Consider this example.  A client recently wanted a private chat system for internal company communications.  The service they had been using wasn't meeting their needs, was littered with bugs, and sometimes didn't work at all.  The core requirement other than privacy, real-time chat, presence, and multi-user chat was that it had to be compliant (all communications stored).  

The learner in me wanted to dive deep, dig into XMPP, build a server from scratch, and accompany that with a web and desktop client.  I spent a few days investigating the technologies involved and even wrote a quick proof of concept (that didn't use XMPP) in PHP.

What I came to realize is that much of the chat landscape had been "solved".  There were rock solid open-source servers that were full-featured, standards compliant, extensible, performant, and scalable (I'm looking at you ejabberd).  In addition, XMPP being such a universally accepted/supported protocol, there were open-source clients for every major OS and even an AJAX web client.

I really did want to write my own XMPP client and server.  Perhaps I will some day, but only if it solves a problem the business is having that can't be solved through the use of existing tools.  In my opinion this is a reminder to "keep your eye on the prize".  If time and resources are infinite then by all means dig in.  Since in business that's rarely (if ever) the case, it's a good lesson learned.

Ask yourself the question "are we in the business of compliant real-time chat?".  If the answer is no take it off the shelf and solve the problem.
