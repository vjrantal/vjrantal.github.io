---
id: 131
title: Headless Wifi onboarding of Windows 10 IoT Core devices
date: 2016-03-24T12:20:55+00:00
author: Ville Rantala
layout: post
dsq_thread_id:
  - 4689077789
categories:
  - IoT
---
# Introduction

In this post, I&#8217;ll explain and show with screenshots how to onboard a device running [Windows 10 IoT Core](https://dev.windows.com/en-us/iot) onto a Wifi access point. Onboarding here means setting up the Wifi settings on the device so that the device gets connected to the wanted network to allow further work with the device.

The approach in this post is &#8220;headless&#8221;, because it doesn&#8217;t require a monitor to be connected to the device and all the input is done from the host machine (so no need to connect keyboard or mouse to the device).

The headless onboarding is especially useful in an environment where you have many devices (and potentially many developers) and it isn&#8217;t feasible to have a separate monitor and input devices to each of them.

# Required software and hardware

The tool used in this post is the [IoT Dashboard](https://ms-iot.github.io/content/en-US/win10/IoTDashboardTroubleshooting.htm) that can be downloaded from [here](http://go.microsoft.com/fwlink/?LinkID=708576). The tool works on Wifi-capable Windows machines (the tool most likely won&#8217;t work in a virtual machine, because it requires Wifi capabilities in a way that isn&#8217;t available in a virtualized environment).

The Windows IoT Core device needs to have Wifi hardware. On devices like Raspberry Pi 2 one would [use a Wifi adapter](https://ms-iot.github.io/content/en-US/win10/SetupWiFi.htm) and on some devices like Raspberry Pi 3 the Wifi would be built-in.

# Steps

## 1. Verify that the device is found

After flashing and powering on a device, such as a Raspberry Pi 2, the tool should show the device in the &#8220;My devices&#8221;-tab. The name of the devices initially comes from the Wifi access point name that the device &#8220;advertise&#8221; for onboarding.

<a href="{{site.baseurl}}/images//uploads/2016/03/1-starting-point.png" rel="attachment wp-att-133"><img class="alignnone size-full wp-image-133" src="{{site.baseurl}}/images//uploads/2016/03/1-starting-point.png" alt="1-starting-point" /></a>

## 2. Start configuring the device

The device configuration is started by clicking the &#8220;Configure device&#8221;-link. There will be a note like below that informs that you will lose your internet connectivity on the host machine while the onboarding is happening (unless you have internet connectivity via a cable). This is because your host machine needs to temporarily connect directly to the IoT Core device to pass the right setup information.

<a href="{{site.baseurl}}/images//uploads/2016/03/2-configure-device.png" rel="attachment wp-att-134"><img class="alignnone size-full wp-image-134" src="{{site.baseurl}}/images//uploads/2016/03/2-configure-device.png" alt="2-configure-device" /></a>

## 3. Setup Wifi

After your host machine is connected to the IoT Core device, you should see a list of access points that the IoT Core device has access to. Select the one you want (typically the one you will connect also with your host machine).

<a href="{{site.baseurl}}/images//uploads/2016/03/3-setup-wifi.png" rel="attachment wp-att-135"><img class="alignnone size-full wp-image-135" src="{{site.baseurl}}/images//uploads/2016/03/3-setup-wifi.png" alt="3-setup-wifi" /></a>

## 4. Access the device via the configured Wifi

If all goes well, your IoT core device will get connected to the Wifi you selected in the previous step.

If you host machine is also connected to the same network and the network allows routing between peers, you will see your IoT Core device pop up to the &#8220;My devices&#8221;-tab like below. This time, the name comes from the name read from the device and the IP address reflects to the one given by the Wifi router.

&nbsp;

<a href="{{site.baseurl}}/images//uploads/2016/03/5-end-result.png" rel="attachment wp-att-137"><img class="alignnone size-full wp-image-137" src="{{site.baseurl}}/images//uploads/2016/03/5-end-result.png" alt="5-end-result" /></a>

If the IoT Core device doesn&#8217;t appear in the list, you could try rebooting it. The Wifi access point settings are persisted between reboots so there is a good chance that after the reboot, the device gets connected to the right network.

Once you have confirmed the device has an IP address, you can do further work such as:

  * Open the device portal from the tool
  * Create SSH or PowerShell connection to the device
  * Set this IP as the remote device in Visual Studio and deploy apps to it

As the final note, if you are in an environment with tens of devices, it isn&#8217;t necessarily obvious which physical device gets which IP address since there is no UI to look it from. One option would be to shutdown a device with a specific IP address and see visually which one got turned off.

&nbsp;