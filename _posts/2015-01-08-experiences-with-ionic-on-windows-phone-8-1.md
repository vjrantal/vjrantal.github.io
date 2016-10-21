---
id: 32
title: Experiences with Ionic on Windows Phone 8.1
date: 2015-01-08T11:57:23+00:00
author: vjrantal
layout: post
dsq_thread_id:
  - 3512372758
categories:
  - Ionic
  - Web
  - Windows Phone
---
# Introduction

In this blog post, I will cover my experiences with the [Ionic Framework](http://ionicframework.com/) on Windows Phone 8.1 while using it in a fairly simple [AllJoyn](https://allseenalliance.org/) [chat app sample](https://github.com/vjrantal/cordova-chat-alljoyn). I&#8217;ll go through the issues I run into and how I solved them.

Ionic is one of the UI framework options for building cross-platform Web apps. During the writing of this post, Ionic doesn&#8217;t yet officially support Windows Phone. It has been [requested in the forums](http://forum.ionicframework.com/t/windows-phone-support/60) and it has been [mentioned](http://forum.ionicframework.com/t/ionic-and-windows-phone/7932/3) to be in the roadmap. Like [others](http://appfoundry.be/blog/2014/10/16/ionic-windows-phone/) have also concluded, even though the official support is not there, the framework can be made usable with some tweaks.

# Scope

This post will focus on the issues I didn&#8217;t find explained elsewhere and ones that exist in the Windows Runtime -based environment on Windows Phone 8.1. These findings are also applicable when running the app on desktop Windows 8.0 or 8.1, because they are also based on the same Windows Runtime environment as what you find on Windows Phone 8.1.

It is important to note that if you use Cordova scripts to manage your project (and add platforms), the difference is that the following command creates the newer [&#8220;universal app platform&#8221;](https://msopentech.com/blog/2014/09/25/apache-cordova-gains-windows-8-1-and-windows-phone-8-1-support-2-2/) which is the one I talk about in this post:

    > cordova platform add windows

The other option is the command below that creates a platform based on the Windows Phone 8.0 Silverlight environment:

    > cordova platform add wp8

An app built on either platform works on Windows Phone 8.1, but due to the differences in the runtime, your app behavior might differ.

# Issues

  * **UI was not appearing to the screen:**

    Windows Runtime has a more restricted policy for injecting content with JavaScript and to allow frameworks like AngularJS to work without modifying them, some custom code is needed. Perhaps the easiest option is to include the [winstore-jscompat.js library](https://github.com/MSOpenTech/winstore-jscompat) that handles the needed modifications. However, after doing this, my app still didn&#8217;t work as expected and the result was that almost nothing got visible to the screen. After some debugging rounds, I found out that the root cause was a bug in the mentioned library, which resulted into extra body elements being created to the DOM tree, which resulted into various issues in rendering the app properly. Luckily, found out that this issue was already found [by others](https://github.com/MSOpenTech/winstore-jscompat/issues/8) and there was a [fix available](https://github.com/ClemMakesApps/winstore-jscompat) that solved my problem. Hopefully the fix gets soon integrated to the master version of the library so that it is enough to include the library as is.

  * **Ionic popovers got rendered to wrong position:**

    This was due to a difference in what document.body.clientWidth returned on IE 11. You can see how I fixed this from the [pull request](https://github.com/driftyco/ionic/pull/2867) I made towards the Ionic project.</li> 
    
  * **Content was not scrollable:**

    The way content zooming works by default on the Windows Runtime -based environment interferes with how content scrolling is implemented in Ionic. I fixed it by adding this to my CSS file:</p> 
        body {
          -ms-content-zooming: none;
        }
    
    This issue is also fixed by a [pending pull request](https://github.com/driftyco/ionic/pull/2518) that contains some more information.
    
  * **ng-click handlers got called twice:**

    Debugging revealed that this issue seems to relate to an incompatibility between the [Ionic&#8217;s tap implementation](https://github.com/driftyco/ionic/blob/ce3aa18018a7f6eb4d271cf75d94fd0a1d986215/js/utils/tap.js) and the [pointer events implementation in IE 11](http://msdn.microsoft.com/en-us/library/ie/dn433244%28v=vs.85%29.aspx). This issue is now reported and a [comment in the report](https://github.com/driftyco/ionic/issues/2885#issuecomment-69006042) shows a workaround that you can apply in your app to get rid of the issue. </li> 
    
  * **App crashes in virtual keyboard use cases:**

    This was because the Ionic project template I used [uses by default](https://github.com/driftyco/ionic-app-base/blob/348ba502e70711a110e02f3143c2b7dcd48c14a4/www/js/app.js#L10-L14) a platform-specific keyboard plugin that is not available yet for Windows Phone. There is a check for the presence of the &#8220;window.cordova.plugins.Keyboard&#8221;-object, but due to the way the [plugin is implemented](https://github.com/driftyco/ionic-plugins-keyboard/blob/d3ee72465b35db7c691d526c4968141c2c63a9cc/plugin.xml#L13-L15), this object is always available &#8211; also on Windows. I ended up removing the plugin and the [usage of the plugin](https://github.com/vjrantal/cordova-chat-alljoyn/commit/6216e31c261900d4b289086d4e2090aef8a1405c), but if you need it on other platforms, a better would be to add a Windows-specific condition to the if-statement. So, something like:

```
-    if(window.cordova && window.cordova.plugins.Keyboard) {
+    if (window.cordova && window.cordova.plugins.Keyboard && !ionic.Platform.isWindowsPhone()) {
```

Good solution in the future would be to include Windows Phone support to the Ionic keyboard plugin.