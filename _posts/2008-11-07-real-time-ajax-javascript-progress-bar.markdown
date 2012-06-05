---
date: '2008-11-07 13:05:19'
layout: post
slug: real-time-ajax-javascript-progress-bar
status: publish
title: Real-time "AJAX" JavaScript Progress Bar
wordpress_id: '43'
categories:
- Development
- JavaScript
- PHP
tags:
- ajax
- iframe
- JavaScript
- long polling
- php
- progress bar
---

UPDATE 2011-01-23: I've revisited this old trick and fixed some bugs some of which were in the comments, so thanks.  The changes are the setTimeout in the parent window's update function (to give the DOM some time to render) and the use of str_pad to force the browser to begin parsing/execution.

I've been developing a stock market screener with some advanced functionality.  Some of the parameters a user might enter, because of their complexity or sheer volume of data required to calculate, would cause the screener to run for considerable amounts of time before returning results.  The opportunity to create a progress bar presented itself.  **EVERY DEVELOPER LOVES CREATING PROGRESS BARS**.

The technique is fairly straightforward.  Here's an overview:



	
  1. Dynamically create a hidden IFRAME

	
  2. Post to the IFRAME

	
  3. Backend script outputs (in real-time) individual <SCRIPT> tags to the IFRAME.

	
  4. Each <SCRIPT> tag contains a single function call to the parent of the IFRAME ( parent.myFuncName(); )

	
  5. Parent JavaScript function updates the status bar with newly passed parameters


A bit of background information.  The "processing" thead is initiated when a user hits the 'Screen' button.  It's an AJAX request to a PHP backend.  While executing, for each iteration, it sets status variables in memcache.  These are the variables that our "status" thread will be able to fetch.

_**Note: **There are a variety of ways I can think of to use this same technique to achieve a real-time progress bar satisfying different situations.  For example, your "processing" thread can be your "status" thread if, for each iteration, it outputs the necessary calls we'll discuss below.  That would allow you to avoid the situation of delivering status data to a seperate thread via memcache (or some other technology)._

I previously had been using an AJAX long-polling technique to achieve this.  If you're not familiar with AJAX long-polling it's essentially when you make a subsequent AJAX request on completion of the prior request to achieve a simulated, continual, "stream" from a server.  The problem in using this technique for a progress bar is two-fold:



	
  1. Multiple, continual, repeated requests to a web server.

	
  2. The data "stream" is pseudo real-time and is affected by variations in each request's latency.  Not very pretty.


All examples below utilize the Prototype JavaScript library.

**Create the hidden IFRAME**
```jscript
// create status iframe
var statusFrame = new Element('iframe', { id: 'statusFrame', name: 'statusFrame' }).hide();
$$('body')[0].appendChild(statusFrame);
```

**Post to the IFRAME**
```jscript
// create status form
var statusForm = new Element('form', { action: '/stocks/screenstatus', method: 'post', target: 'statusFrame' });
$$('body')[0].appendChild(statusForm);

// post to iframe
statusForm.submit();
```

**Backend script snippet to output <SCRIPT> tags**
```php
public function screenstatus()
{
        // pad to force the browser to starting parsing/executing
        echo str_pad('<html><body>', 4096);
	while(1) {
		//status string
		$status = Mcache::get('status');
		
		// how many have been processed
		$c = Mcache::get('c');
		
		// how many results
		$rc = Mcache::get('rc');
		
		// total
		$t = Mcache::get('t');
		
		echo str_pad('<script type="text/javascript">parent.updateStatus("'.$status.'", '.(int)$c.', '.(int)$rc.', '.(int)$t.');</script>'."\n", 1024);
		flush();
		
		if(($status === false) || ($status === 'canceled') || ($status === 'complete')) {
			break;
		}
		
		usleep(25000);
	}

        echo '</body></html>';
}
```

**Parent JavaScript function to update progress bar**

I chose to use a div with a background color and a dynamically adjusted width as the visual element for my progress bar.  Initially the width is set to 0.  In each updateStatus() call the width is adjusted to the current % of the whole (which in my case is 675px, the final desired width of the progress bar).

To overlay text I have a 2nd div styled 'position: relative;' with negative 'top' and 'bottom-margin'.  This positions the textual div on top of the progress bar div.
```css
#statusProgressBar {
width: 0px;
height: 29px;
background: #f4f4f4;
}
#statusProgress {
position: relative;
top: -29px;
left: 0;
text-align: center;
width: 675px;
padding-top: 3px;
height: 26px;
margin-bottom: -29px;
color: #00a;
font-size: 10px;
}
```
```html
<div id="statusProgressBar"></div>
<div id="statusProgress"></div>
```
```jscript
function updateStatus(status, c, rc, t)
{
	var statusHTML;
	var progressBarWidth;
	
	if(status == 'initializing') {
		$('statusProgressBar').setStyle({ width: '0px' });
		statusHTML = '<span id="top">Initializing...</span>';
	} else if(status == 'canceled') {
		$('statusProgressBar').setStyle({ width: '0px' });
		statusHTML = '<span id="top">Canceled... '+number_format(c / t * 100, 2)+'% Complete</span>';
	} else {
		if(t) {
			statusHTML = '<span id="top">'+rc+' result(s) ('+number_format((c ? (rc / c) : 0) * 100, 2)+'%)</span><br/><span id="bot">'+c+' of '+t+' processed ('+number_format(c / t * 100, 2)+'%)</span>';
		} else {
			statusHTML = '<span id="top">0 result(s)</span>';
		}
		progressBarWidth = Math.floor(675 * (c / t));
		$('statusProgressBar').setStyle({ width: progressBarWidth+'px' });
	}
	
        // give the DOM some time to actually render
        setTimeout(function() { $('statusProgress').update(statusHTML); }, 10);
}
```

You'll notice that the updateStatus() function calls number_format().  It's functionally equivalent to PHP's number_format().  Here is the JavaScript code below:
```jscript
function number_format( number, decimals, dec_point, thousands_sep ) {
    // http://kevin.vanzonneveld.net
    // +   original by: Jonas Raoni Soares Silva (http://www.jsfromhell.com)
    // +   improved by: Kevin van Zonneveld (http://kevin.vanzonneveld.net)
    // +     bugfix by: Michael White (http://getsprink.com)
    // +     bugfix by: Benjamin Lupton
    // +     bugfix by: Allan Jensen (http://www.winternet.no)
    // +    revised by: Jonas Raoni Soares Silva (http://www.jsfromhell.com)
    // +     bugfix by: Howard Yeend
    // *     example 1: number_format(1234.5678, 2, '.', '');
    // *     returns 1: 1234.57     
 
    var n = number, c = isNaN(decimals = Math.abs(decimals)) ? 2 : decimals;
    var d = dec_point == undefined ? "." : dec_point;
    var t = thousands_sep == undefined ? "," : thousands_sep, s = n < 0 ? "-" : "";
    var i = parseInt(n = Math.abs(+n || 0).toFixed(c)) + "", j = (j = i.length) > 3 ? j % 3 : 0;
    
    return s + (j ? i.substr(0, j) + t : "") + i.substr(j).replace(/(\d{3})(?=\d)/g, "$1" + t) + (c ? d + Math.abs(n - i).toFixed(c).slice(2) : "");
}
```

I hope you found some of this code useful.  I welcome comments and criticism!
