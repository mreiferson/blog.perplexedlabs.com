---
date: '2011-05-16 09:00:29'
layout: post
slug: experiences-developing-my-first-ios-iphone-app-oncall-for-nagios
status: publish
title: Experiences Developing My First iOS / iPhone App - OnCall for Nagios
wordpress_id: '517'
categories:
- Development
- Infrastructure
- iOS
tags:
- ios
- ios sdk
- ipad
- ipad sdk
- iphone
- iphone sdk
- nagios
- oncall
---

I've long since wanted to participate in the explosion of popularity that is the iOS App Store.  I toyed with demonstration-purpose applications, learning the foundational aspects of developing on this new platform and the intricacies of Objective-C, but never really had a compelling idea to follow through to completion.

Due to the nature of my current employment I spend alternating periods of time as primary oncall responder when issues arise in our infrastructure.  I've long been a fan of [Nagios](http://www.nagios.org/), which we use at work as well.  It lacks a native iPhone interface, but provides easy-enough hooks to be able to build one.  This gave me a fantastic opportunity to hone my iOS SDK skills with a app that would certainly "scratch my own itch".  I'm a strong believer that you learn best when addressing a problem you're actually experiencing - you can taste the pain going away.  With **[OnCall for Nagios](http://bit.ly/nagiosss)**, the iPhone becomes a fantastic platform to be able to do basic triage and diagnosis as well as respond and communicate to other participants of your infrastructure team.

The great thing about this as my first _real_ application was that it required a deeper dive into many of the core iPhone SDK APIs as well as the building of a more complicated (from a development perspective) UI.

Under the hood it's interaction with Nagios is accomplished via screen-scraping.  This allows the app to work right out of the box, acting as just another client to the existing Nagios web interface without any server side changes required.  For HTML parsing I built a light abstraction around libxml and used xpath queries to retrieve the relevant data.  Certain views required multiple asynchronous HTTP requests, each with the same delegate, which required another light abstraction layer around NSURLConnection to be able to pass in identifiers for the connections so that incoming data would be appended to the correct NSMutableData property.  Seemingly simple tasks such as HTTP authentication for async NSURLConnection requests and (optionally) ignoring self-signed SSL certificates took significant digging through documentation to gain a better understanding of connection's didReceiveAuthenticationChallenge method.

**[OnCall for Nagios](http://bit.ly/nagiosss)** uses a TabBarController with child NavigationBarControllers - this gives you the familiar tabbed buttons at the bottom with the ability to navigate forwards and backwards at the top.  It also uses a combination of custom and NSUserDefaults to provide saved settings - allowing multiple Nagios instances to be setup and switched between within the app.  







[![Problems View Screen Shot](http://blog.perplexedlabs.com/wp-content/uploads/2011/05/sshot4.png)](http://blog.perplexedlabs.com/wp-content/uploads/2011/05/sshot4.png)






[![Service Detail View](http://blog.perplexedlabs.com/wp-content/uploads/2011/05/sshot5.png)](http://blog.perplexedlabs.com/wp-content/uploads/2011/05/sshot5.png)










Interface development was slow and tedious for the initial setup.  Determining the correct nesting order (TabBarController is the parent of multiple NavigationBarControllers) was huge.  Once you get the hang of IBOutlets and where to "connect the dots" duplicating previous discoveries becomes easier.  For certain interface details I often found myself hand-coding these elements instead of arguably spending more time figuring out how to do them correctly in Interface Builder.  I'd be curious to hear of other's experiences on where to draw that line.  I found that anything non-trivial required some level of code (take custom UITableViewCells) although I'm sure there are other ways to accomplish this.

One of the most important aspects to developing in Objective-C is a solid of understanding of it's memory management.  With a solid background in C as well as Python - this wasn't terribly difficult.  **[This document](http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/MemoryMgmt/MemoryMgmt.html)** was extremely helpful.  If you follow the golden rule **"you only release or autorelease objects you own"** you'll be fine.  I found Xcode's static code analysis (clang) to be extremely helpful in situations that weren't immediately obvious, diagnosing and eliminating problems in my code.  It's quite impressive to see arrows being overlaid onto your source code illustrating the code path that's causing the issue.

[![Enable Static Code Analyzer](http://blog.perplexedlabs.com/wp-content/uploads/2011/05/clang3.png)](http://blog.perplexedlabs.com/wp-content/uploads/2011/05/clang3.png)

I also highly recommend [Flurry](http://www.flurry.com), a free analytics SDK.  It's extremely easy to integrate and provides a wide variety of metrics without much effort.  It was also helpful in identifying when Apple was reviewing the application.  The approval process took longer than I had hoped but wasn't as problematic as I had imagined it to be.

I also found it extremely beneficial to release a _lite_ version.  For this type of application the choices were obvious as to limiting functionality (display fewer problems, dont allow multiple instances, only show 2 hosts, etc.).  In terms of implementation of the lite version - it exists in the same codebase.  In Xcode I defined a new target with a different output binary and a specific C define which the code pivots on.

```c
#ifdef LITE_VERSION
    if ([table count] < 2) {    
        [table addObject:data];
    } else if (!didShowUpgrade) {
        didShowUpgrade = TRUE;
        UIAlertView *alertBox = [[UIAlertView alloc] initWithTitle:@"Lite Version" message:@"The lite version only displays 2 problems, consider upgrading!" delegate:self cancelButtonTitle:@"Ok" otherButtonTitles:nil];
        [alertBox show];
        [alertBox release];
    }
#else
    [table addObject:data];
#endif
```

The lite version sees much higher downloads and facilitates the need for experimentation and testing before purchasing.

I've certainly learned a lot throughout this process and I'm looking forward to continuing to develop features for this app as well as getting back into game development on this powerful platform.  If you'd like more detail on anything I've glossed over feel free to ask questions in the comments or e-mail me directly.

Good luck!
