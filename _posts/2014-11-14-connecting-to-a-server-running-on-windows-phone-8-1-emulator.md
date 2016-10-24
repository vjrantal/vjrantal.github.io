---
id: 21
title: Connecting to a server running on Windows Phone 8.1 emulator
date: 2014-11-14T13:07:25+00:00
author: Ville Rantala
layout: post
dsq_thread_id:
  - 3521383533
categories:
  - Windows Phone
---
The networking configuration of the latest Windows Phone 8.1 emulator has changed from the Windows Phone 8.0 emulator and this has some implications to how you can connect to and from the emulator.

In this post, I&#8217;ll describe what I had to do in a case where I had a server running inside the emulator and I wanted to create a connection to it. I am using a [simplistic server app](http://developer.nokia.com/community/wiki/A_simplistic_HTTP_Server_on_Windows_Phone) as the example.

If you run the example app on a Windows Phone 8.0 emulator, you can access the server using the IP address that the app prints. A screenshot (with IE on the background and emulator on the foreground) would look something like this:

[<img class="alignnone size-full wp-image-24" alt="wp80-emulator" src="{{site.baseurl}}/images/uploads/2014/11/wp80-emulator.png" />]({{site.baseurl}}/images/uploads/2014/11/wp80-emulator.png)

However, doing the same with a 8.1 emulator doesn&#8217;t seem to work and the browser can&#8217;t access the server. Instead of using the IP the app prints, I had to use the IP of the Hyper-V Virtual Ethernet Adapter. Here is a screenshot of that case &#8211; **notice how the IP in the browser address bar differs from what the app prints**:

[<img class="alignnone size-full wp-image-25" alt="wp81-emulator" src="{{site.baseurl}}/images/uploads/2014/11/wp81-emulator.png" />]({{site.baseurl}}/images/uploads/2014/11/wp81-emulator.png)

There are at least two places from which you can see the IP address to connect to:

**1. Network and Sharing Center**

[<img class="alignnone size-full wp-image-26" alt="network-connection-details" src="{{site.baseurl}}/images/uploads/2014/11/network-connection-details.png" />]({{site.baseurl}}/images/uploads/2014/11/network-connection-details.png)

**2. Emulator&#8217;s tools menu under the network tab**

[<img class="alignnone size-full wp-image-27" alt="emulator-tools" src="{{site.baseurl}}/images/uploads/2014/11/emulator-tools.png" />]({{site.baseurl}}/images/uploads/2014/11/emulator-tools.png)