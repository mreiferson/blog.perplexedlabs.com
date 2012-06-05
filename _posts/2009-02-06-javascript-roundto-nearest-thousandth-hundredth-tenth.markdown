---
date: '2009-02-06 13:53:42'
layout: post
slug: javascript-roundto-nearest-thousandth-hundredth-tenth
status: publish
title: JavaScript roundTo Nearest Thousandth, Hundredth, Tenth, *
wordpress_id: '153'
categories:
- Development
- JavaScript
tags:
- JavaScript
- prototype
- round
- rounding
---

This very simple function does exactly what the title suggests, it allows you to round to any specified accuracy.

```javascript
function roundTo(number, to) {
	return Math.round(number * to) / to;
}

alert(roundTo(1532, 100)); // 1500
alert(roundTo(26, 10)); // 30
```

If you want to get fancy, you could also modify the Number prototype for a more "integrated" solution.

```javascript
Number.prototype.roundTo = function(to) {
	return Math.round(this * to) / to;
}
``` 
