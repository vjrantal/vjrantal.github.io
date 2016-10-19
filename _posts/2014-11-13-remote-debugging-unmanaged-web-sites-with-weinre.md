---
id: 11
title: Remote debugging unmanaged Web sites with weinre
date: 2014-11-13T16:02:20+00:00
author: vjrantal
layout: post
dsq_thread_id:
  - 3512354598
categories:
  - Debugging
  - Web
---
[Weinre](http://people.apache.org/~pmuellr/weinre-docs/latest/) can sometimes be a useful tool for remotely debugging Web sites that may be targeted to run on mobile devices. The installation and running is easy and there is [lots of help](https://www.google.com/search?q=weinre) available to do that.

The idea is that you instrument the Web site you want to debug with an initialization script so that a connection is being made between the site (which is the debug target), the weinre debug server (that you can host yourself) and the debugging user interface (Web Inspector). However, there might sometimes be a case where you can&#8217;t modify the site you want to debug. This could be, for example, because you are not the author of the site or you want to debug something in a production environment that you can&#8217;t easily instrument.

When I had a case like this, I setup an HTTP proxy in between the origin of the Web site and the debug target. In the proxy, I rewrote the responses so that the weinre initialization script got injected into the HTML pages. This way, I was able to debug an unmanaged Web site running on iOS, Android or Windows Phone devices using a single debugging user interface.

As the proxy, I run Charles on my Mac and used the [rewrite tool](http://www.charlesproxy.com/documentation/tools/rewrite/) to modify the responses. The rule I had in place looked like this:

[<img class="alignnone size-full wp-image-17" alt="Weinre rewrite rule" src="http://blog.vjrantal.net/wp-content/uploads/2014/11/Screen-Shot-2014-11-13-at-15.51.15.png" width="846" height="554" />](http://blog.vjrantal.net/wp-content/uploads/2014/11/Screen-Shot-2014-11-13-at-15.51.15.png)

Another alternative for the proxy is Fiddler, which also seems to have similar [capability in place](http://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse).

This way, after I setup my mobile devices to use my HTTP proxy, all the sites I visited had the weinre debugging enabled. Since weinre works nowadays also with non-WebKit browsers like [on Windows Phone](http://msopentech.com/blog/2013/05/31/now-on-ie-and-firefox-debug-your-mobile-html5-page-remotely-with-weinre-web-inspector-remote/), I was able to easily compare the site behavior on multiple devices.

As an additional configuration, I had [SSL proxying](http://www.charlesproxy.com/documentation/using-charles/ssl-certificates/) enabled in Charles to be able to debug SSL-protected sites.