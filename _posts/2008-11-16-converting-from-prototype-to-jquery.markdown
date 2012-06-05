---
date: '2008-11-16 21:52:47'
layout: post
slug: converting-from-prototype-to-jquery
status: publish
title: Converting from Prototype to jQuery
wordpress_id: '68'
categories:
- Development
- JavaScript
tags:
- framework
- JavaScript
- jquery
- prototype
---

Let me start off by saying I absolutely love Prototype.  When a friend showed me how much it helped with writing cross-browser compatible code, I was an instant fan.

Over the years of being a Prototype user I paid close attention to the many posts about why X is a better framework than Y.  I had always planned on at least TRYING another framework at some point to see what all the fuss was about.

Initially what attracted me to jQuery was it's compactness.  It doesn't have separately defined classes for each component, like Prototype.  There's one point of entry and most everything revolves around DOM elements.  There's no distinction between selecting a single or multiple DOM element(s) and, because jQuery only uses a selector syntax, you don't get confused whether the parameter passed is an ID, class, etc.

Furthermore, I never really liked having to use $ for single selection and $$ for multiple selections in Prototype.  Relatively speaking, with jQuery I find that performing actions on multiple DOM elements is easier.  

Chaining is really where jQuery shines.  It reminds me of using the 'with' construct in Visual Basic back in the day.  "This is the object that I want to perform actions on... do this, that, and this... thanks!".  

However, unlike Prototype where one could test for the existence of an object by:

```jscript
if($('objectID')) {
// do something
}
```

In jQuery it isn't quite so intuitive.  Since all jQuery instances are collections of elements, you test for the existence of an element by checking its length property:

```jscript
if(jQuery('#objectId').length == 1) {
// do something
}
```

In jQuery, binding to events is also very intuitive.  Combined with the power of chaining adding event handling to multiple objects is ridiculously simple.  And, because the code executed in your event callback has the context of the object that triggered the event, you can (more easily than bindAsEventListener in Prototype) just access the object within the callback via 'this'.

This brings me to one issue that did initially cause some conversion conflict.  Prototype does a good job of automatically extending standard objects, objects returned, and parameters passed.  With jQuery, you have to get used to 'get()'ing the root DOM element if you need to manipulate it and likewise, extending a root DOM element to a full fledged jQuery instance (such as 'jQuery(this)') if you want to manipulate it via jQuery.

Another primary reason why I switched to jQuery is because it has built in support for basic animations.  Transitional animations give that extra something to a web app.  Having to hand-code them isn't pleasant.  Obviously Prototype has scriptaculous, but thats a huge size (and therefor performance) commitment to make for just a simple fade, no?

jQuery was built to be extensible.  It seems to me that Prototype intended to be the "be all, end all" solution to your JavaScript woes.  jQuery takes a more sensible approach - it provides the fundamental structure and handles the most-desired scenarios.  If there's something it doesn't do out of the box there's probably a plugin available that does.  Most likely someone has come across the same shortcoming already and written a plugin to fulfill the need.

Some plugins I ended up needing immediately were:




	
  * Date picker

	
  * CSS Extraction from Ajax responses (IE issue)

	
  * Ancestry (to mimic Prototype's descendantOf())

	
  * DOM element creation (to mimic the functionality of Prototype's Element class)



I prefer to create DOM elements in JavaScript programatically without using HTML.  jQuery doesn't have a built in solution for creating DOM elements like Prototype does with its Element class.  Fortunately, Lukasz Rajchel has written the DOMEC (DOM Element Creation) plugin.  It's lightweight and is a drop-in replacement for those familiar with the syntax of Prototype's Element.

During the conversion process I found it to be most helpful to load jQuery side by side with Prototype using jQuery's no-conflict functionality.  As I slowly combed through my JavaScript removing Prototype dependencies I would periodically remove Prototype to see if and where I missed any outliers.  This also requires you to use the jQuery() syntax as opposed to the traditional $() syntax.  In doing so this helps easily identify where Prototype code remains.

How have your conversions to jQuery gone?  Have you converted and never looked back?  Your thoughts are appreciated...
