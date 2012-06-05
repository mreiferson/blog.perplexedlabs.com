---
date: '2008-11-12 10:34:17'
layout: post
slug: javascript-event-handler
status: publish
title: JavaScript Event Handler
wordpress_id: '65'
categories:
- Development
- JavaScript
tags:
- event handler
- JavaScript
---

**2008-11-16: Updated the source code to remove the Prototype dependency**

This is a very simple, lightweight, object to manage the handling of events in JavaScript.  Events can be arbitrarily defined and identified based on calls to listen().  Multiple actions can be handled per event and events can be triggered arbitrarily at any point in your code with calls to trigger().

```jscript
var EventHandler = {
	events: [],
	actions: [],
	index: {}
};

EventHandler.listen = function(evt, action)
{
	var idx = this.events.length;
	
	// add a new event entry if one doesn't exist already
	if(typeof(this.index[evt]) == 'undefined') {
		this.events[idx] = evt;
		this.index[evt] = idx;
		this.actions[idx] = [];
	}
	
	// add to the list of actions for this event
	this.actions[idx][this.actions[idx].length] = action;
};

EventHandler.trigger = function(evt, args)
{
	var idx = this.index[evt];
	
	if(typeof(idx) != 'undefined') {
		// cycle and call the actions for this event
		for(var i = 0, len = this.actions[idx].length; i < len; ++i) {
			action = this.actions[idx][i];
			action(args);
		}
	}
};

/*
EventHandler.listen('test', function() { alert('Testing Event Handler'); });
EventHandler.trigger('test');
*/
``` 
