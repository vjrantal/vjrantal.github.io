---
id: 74
title: Building a Cordova plugin including native code for Windows platform
date: 2015-03-12T14:49:18+00:00
author: vjrantal
layout: post
dsq_thread_id:
  - 3589049274
categories:
  - Cordova
  - Web
  - Windows
  - Windows Phone
---
# Introduction

In the Cordova world, [plugins](https://cordova.apache.org/docs/en/edge/guide_hybrid_plugins_index.md.html) are the way to expose features from the native platforms to the JavaScript environment the app runs on. In this post, I&#8217;ll explain how we built [a plugin](https://github.com/AllJoyn-Cordova/cordova-plugin-alljoyn/) that includes a native library (written in C) and can that be built for Windows and Windows Phone using &#8220;standard&#8221; Cordova tools.

The post first talks about how to structure and define the plugin. Then, it includes notes about building and running and finally lists some Windows-specific caveats we run into.

Already for some time, there has been [support](https://msopentech.com/blog/2014/09/25/apache-cordova-gains-windows-8-1-and-windows-phone-8-1-support-2-2/) for writing plugins for Windows in a way that it works on both Windows (>= 8.0) and Windows Phone (>= 8.1). However, only recently, a [Cordova release](http://cordova.apache.org/news/2015/03/02/tools-release.html) was made that improved the features on this area. In particular, [this feature](https://issues.apache.org/jira/browse/CB-8123) helps defining a plugin in a way it can be built to Windows and Windows Phone using the Cordova command-line interface (kudos for Tim Barham for the [implementation](https://github.com/apache/cordova-lib/pull/164)).

# Plugin definition

The Windows-specific part of our plugin&#8217;s [plugin.xml](https://github.com/AllJoyn-Cordova/cordova-plugin-alljoyn/blob/master/plugin.xml) file looks like this:



Here is some further explanation of the snippet of XML above:

  * On line 5, we describe that to build for Windows, our plugin requires at least version 3.8.0 of the Cordova &#8220;engine&#8221;, because of the feature mentioned in the introduction section
  * Lines 8 to 10 define the file that contains the &#8220;public API&#8221; of our plugin
  * The Windows-specific &#8220;proxy&#8221; JavaScript code is pointed at line 15
  * Lines 18 and 19 are the pointers to the projects that define the native parts of the Windows implementation

So, the architecture of the plugin is very similar that is instructed in the Cordova project&#8217;s [official 4.0.0 documentation](http://cordova.apache.org/docs/en/4.0.0/guide_platforms_win8_plugin.md.html#Windows%20Plugins), but on lines 18 and 19, you can see the usage of the new feature. The reason there is a need for two project references instead of one is that the current tooling requires building and packaging for Windows and Windows Phone separately. When you use the common API set, you can implement apps (and plugins) universally, but SDK toolchain differs based on the target. You get the idea when you look at the difference between the project files in our case:



In addition to the two target-specific project files, we have a [shared project file](https://github.com/AllJoyn-Cordova/cordova-plugin-alljoyn/blob/master/src/windows/AllJoynWinRTComponent/AllJoynWinRTComponent.Shared.vcxitems), which is overall matching to a project structure you would get if you would create an universal app from Visual Studio project templates.

# Building and running

As mentioned, the plugin can be added, built and run using the Cordova command-line interface (and any tooling that is built on top of that). As an example, here are some commands that one could use after adding the our plugin:

```bash
$ cordova platform add windows
// To run on Windows Phone 8.1 emulator
$ cordova run windows --emulator --archs="x86" -- -phone
// Running on Windows Phone 8.1 device
$ cordova run windows --device --archs="arm" -- -phone
// To run on desktop (current default is Windows 8.1 build)
$ cordova run windows --device --archs="x64" -- -win
```

In our case, it is mandatory to pass a suitable target architecture with the &#8211;archs argument, because otherwise, the default AnyCPU is used (and one can&#8217;t use that with our C library).

For development and debugging workflow, it is often more useful to run the app from Visual Studio so that you can leverage the debugging capabilities. To do that, after the platform is added, you can find a solution file from platforms/windows/CordovaApp.sln that you can open in Visual Studio. In our case, the solution opened looks like this:

<div id="attachment_92" style="width: 357px" class="wp-caption alignnone">
  <a href="{{site.baseurl}}/images/uploads/2015/03/project-structure.png"><img class="size-full wp-image-92" src="{{site.baseurl}}/images/uploads/2015/03/project-structure.png" alt="Solution structure" width="347" height="583" /></a>
  
  <p class="wp-caption-text">
    Solution structure
  </p>
</div>

In above screenshot, you see how there are two Windows Runtime Component projects in the solution that were the ones we pointed to in our plugin.xml and that contain the native parts (including the C library).

Similarly than with the Cordova command-line interface, in Visual Studio you must explicitly change the target architecture from the default before running the app. For example, if you would like to run on a Windows Phone device, you selection would look something like this:

<div id="attachment_93" style="width: 783px" class="wp-caption alignnone">
  <a href="{{site.baseurl}}/images/uploads/2015/03/target-architecture-selection.png"><img class="size-full wp-image-93" src="{{site.baseurl}}/images/uploads/2015/03/target-architecture-selection.png" alt="Target architecture selection" width="773" height="224" /></a>
  
  <p class="wp-caption-text">
    Target architecture selection
  </p>
</div>

# Windows-specific caveats

  * If you have an API that returns JavaScript arrays and you create the arrays in the Windows Runtime side, you may want to consider an additional type conversion since Windows Runtime arrays are not exactly the same as JavaScript arrays. For other platforms, the Cordova &#8220;bridge layer&#8221; takes care of this, but not for Windows.
  * <del>The Cordova exec proxy API is based on passing success and error callback functions as first and second parameter. On other platforms, one can choose to call the success callback several times, but on Windows, you can only call the success callback once. We worked around this by passing the callback function as an additional parameter and using that as the success callback in cases where we knew multiple calls is expected (like in case of event listeners).</del> Update: [There is now an option](https://github.com/sgrebnov/cordova-js/commit/a9371e5959223bfeef163c034baaf0ec0ae597d9) to keep the callbacks also on Windows.
  * The referenced projects must have project configuration for Win32 platform even if you set the target architecture to x86 (not Win32) with the Cordova scripts. There is probably some mapping inside Cordova implementation from x86 to Win32 since at least our plugin failed to build without Win32 platform configuration.
  * The default debugger type in Visual Studio is &#8220;Script Only&#8221; and that isn&#8217;t always enough to spot issues on the managed or native side. You can change the debugger type by right-clicking the project in the Solution Explorer and selecting a more suitable debugger type. Here is the options you have:

    <div id="attachment_98" style="width: 759px" class="wp-caption alignnone">
      <a href="{{site.baseurl}}/images/uploads/2015/03/debugger-type.png"><img class="size-full wp-image-98" src="{{site.baseurl}}/images/uploads/2015/03/debugger-type.png" alt="Debugger types" width="749" height="435" /></a>
      
      <p class="wp-caption-text">
        Debugger types
      </p>
    </div>

  * This one is not Windows-specific, but wanted to point out that it is fairly easy to get the project into a broken state if folders are removed manually (instead of managing the project entirely with Cordova scripts). Here is an example of a problematic scenario if folders are manually removed:

```bash
$ cordova plugin add org.allseen.alljoyn
$ cordova platform add windows
$ rm -rf platforms
$ cordova platform add windows
...
Plugin "org.allseen.alljoyn" already installed on windows.
...
// Cordova thought the plugin was installed already, because it found windows.json in the plugins folder, but that is not true and at this point, the windows project would be in a broken state.
```
