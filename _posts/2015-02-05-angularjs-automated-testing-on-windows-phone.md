---
id: 49
title: AngularJS automated testing on Windows Phone
date: 2015-02-05T14:24:27+00:00
author: Ville Rantala
layout: post
dsq_thread_id:
  - 3512353140
categories:
  - Testing
  - Web
  - WebDriver
  - Windows Phone
---
# Introduction

In this blog post, I&#8217;ll show how one can execute the automated tests of an app written on top of AngularJS on a Windows Phone 8.1 device. If you are only interested in the &#8220;beef&#8221;, please check the &#8220;Setting up&#8221; and &#8220;Running tests&#8221; from below.

To give an idea what the end result could look like, here is a short video of running the [Ionic Framework&#8217;s](http://ionicframework.com/) tests:

<iframe width="560" height="315" src="https://www.youtube.com/embed/juU2GHyCOJc" frameborder="0" allowfullscreen></iframe>

There are many different testing frameworks out there for various types of tests, but under the hood, they often rely on the [WebDriver](http://docs.seleniumhq.org/projects/webdriver/) API for actually controlling or &#8220;driving&#8221; the Web browser. The API abstracts the differences between various browser implementations and by implementing a browser-specific driver, one can get an arbitrary browser hooked into this ecosystem of WebDriver-based testing frameworks.

It is known that Windows Phone doesn&#8217;t have as good support for these frameworks as iOS and Android, but there are solutions one can leverage when testing on a Windows Phone is required.

I think at least two projects are worth mentioning. The first one can be found [here](https://github.com/forcedotcom/windowsphonedriver) and the second one from [here](http://winphonewebdriver.codeplex.com/). Both leverage &#8220;[automation atoms](https://code.google.com/p/selenium/wiki/AutomationAtoms)&#8221; from the Selenium project and execute the tests within an embeddable [WebBrowser component](https://msdn.microsoft.com/query/dev14.query?appId=Dev14IDEF1&l=EN-US&k=k(Microsoft.Phone.Controls.WebBrowser);k(TargetFrameworkMoniker-WindowsPhone,Version%3Dv8.1);k(DevLang-csharp)&rd=true) in a Windows Phone Silverlight app. The main architectural difference is that in the first one, the driver server is implemented as a command-line executable (that is typically run on a Windows desktop machine) whereas in the latter, the server is implemented in the driver app itself.

In this blog post, I&#8217;ll concentrate on the [latter](http://winphonewebdriver.codeplex.com/), because I preferred a solution where I can run the tests from any machine including Mac and Linux (currently, the setting up phase still requires a Windows OS and some developer tools to be able to deploy a XAP file to your device).

# Setting up

  1. **Download the app XAP file from <http://winphonewebdriver.codeplex.com/downloads/get/1426521> and install it to your Windows Phone 8.1 device.** If you are not already a Windows Phone developer and don&#8217;t have the tools installed, this is by far the most time-consuming step, because it requires some amount of [software and tooling](https://msdn.microsoft.com/en-us/library/windows/apps/ff402565%28v=vs.105%29.aspx). If you don&#8217;t have a Windows machine, you can also install the required tools onto a virtual Windows (using for example VMware, Parallels or VirtualBox). If you search for instructions related to setting up a Windows Phone SDK inside a virtual machine, the most complication comes from getting the emulator running, but all of that can be skipped in this context if the WebDriver app is wanted to be deployed onto a real device.
  2. **Open the installed app and take a note of the IP address the device has** like shown in the picture below:

<div id="attachment_58" style="width: 930px" class="wp-caption alignnone">
  <a href="{{site.baseurl}}/images/uploads/2015/02/windows-phone-driver.png"><img class="size-full wp-image-58" alt="Driver settings" src="{{site.baseurl}}/images/uploads/2015/02/windows-phone-driver.png" width="920" height="700" /></a>
  
  <p class="wp-caption-text">
    Driver settings
  </p>
</div>

# Running tests

  1. **Configure the tests to use the remote WebDriver protocol.** How to do this depends on which framework and language bindings you are using, but I am giving some examples below.
  2. **Run the tests and watch them go.** If running fails, one common issue to check here is that the machine that runs the tests and the device is connected to the same network and can access each other. Problems may occur, for example, in case the network has restrictions on how clients can connect to each other or if the machine that runs the tests has a firewall blocking connections.

# Example using angular-seed

The [angular-seed](https://github.com/angular/angular-seed) project serves as a good example since it has both unit tests and end to end tests. Unit tests are written in [Jasmine](http://jasmine.github.io/) and run with [Karma Test Runner](http://karma-runner.github.io/). The end to end tests are using [Protractor](https://github.com/angular/protractor).

**The summary of the changes needed is:**

  * Change the package.json to include [karma-webdriver-launcher](https://github.com/karma-runner/karma-webdriver-launcher) which knows how to run tests against any driver server implementing the API
  * Update karma.conf.js to use our custom launcher and configure it so that karma finds the Windows Phone device
  * For protractor tests, update protractor.conf.js for correct driver IP and the IP of the machine on which we run the tests (so that the phone is able to access the server we run)

In my environment, my Mac had IP address 10.0.1.2 and my Windows Phone had 10.0.1.3. I had commit de30ee955c55ddf27b8fd15789ad18206bbbc285 checked out from angular-seed Here is the full diff of changes I had to make:

After that, I was able to run the tests from my Mac as [instructed](https://github.com/angular/angular-seed/blob/master/README.md). Full output from the commands at:

# Caveats

## Events are simulated with JavaScript

Especially in the end to end tests, you want to simulate a scenario where an end user would actually use an app as closely as possible. On a full-touch Windows Phone device, this is mostly touch-based interactions with fingers. However, because the WebDriver is implemented in the JavaScript layer, the browser engine doesn&#8217;t interpret interactions with the app as touch input and thus the events your app gets differs between test scenario and real usage scenario.

As you can see for example [here](http://patrickhlauke.github.io/touch/tests/results/), the exact behavior (order, timing, etc.) of generated events differs quite a bit between various browser implementations. This may cause end-user-visible issues that would be nice to catch with end to end tests, but unfortunately the WebDriver implementation covered here can&#8217;t be reliably used from this perspective due to the reason mentioned above.

## Run in WebBrowser in Silverlight app

Depends a bit what is the target for your app, but a thing to note is that there might be small differences between how the embeddable WebBrowser within a Silverlight app behaves versus the IE browser on the device versus a JavaScript universal app.

## Not supporting the entire WebDriver specification

The driver doesn&#8217;t support all the functionality defined in the [WebDriver Wire Protocol specification](https://code.google.com/p/selenium/wiki/JsonWireProtocol). This means that depending on which language binding and which exact features you use, you may end up getting an &#8220;not implemented&#8221;-response from the driver.

## Unproven reliability

Based on the data I found online, this driver hasn&#8217;t been in wide use in real-world testing scenarios. This means that there is not much data around how reliable the solution is in complex scenarios and perhaps when hooked into a CI system for continuous testing.